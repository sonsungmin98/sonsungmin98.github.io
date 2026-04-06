---
title: "개인 프로젝트"
date: 2026-04-06
categories: [Portfolio, Project]
tags: [portfolio, unity, ai, combat, ui, tool, addressables]
---

# 개인 프로젝트

최근 작성한 Unity 클라이언트 코드들을 기준으로 정리한 문서입니다.
현재 프로젝트에서는 Character 구조, Behavior Tree 기반 적 AI, 게임 부트와 매니저 구성, UI 프레임워크를 중심으로 작업했습니다.

## Character 구조

캐릭터는 `Character`를 중심으로 구성됩니다.
`Character`는 단순히 플레이어나 적의 공통 부모 클래스가 아니라, 실제 게임 오브젝트가 가져야 하는 기능과 컨트롤러를 묶는 조립 지점 역할을 합니다.

```c#
public sealed class Character : FieldObject
{
    [SerializeField] private MonoBehaviour controllerBehaviour;
    private IController? _controller;

    [SerializeField] private AnimatorBridge animBridge;
    [SerializeField] private MovementDriver move;

    public FactionFeature Faction => GetFeature<FactionFeature>();
    public StatsFeature Stats => GetFeature<StatsFeature>();
    public StatusFeature Status => GetFeature<StatusFeature>();
    public LifeFeature Life => GetFeature<LifeFeature>();
    public AttackEmitterFeature AttackEmitter => GetFeature<AttackEmitterFeature>();
    public ActionRunnerFeature Actions => GetFeature<ActionRunnerFeature>();
}
```

### Feature 기반 구조
캐릭터의 실제 기능은 `BuildFeatures()`에서 조립합니다.
팩션, 스탯, 상태, 생명, 무기, 공격 판정, 액션 실행, 소울 스택, 대시 획득 같은 기능을 각각 Feature 단위로 나눠 두었습니다.

```c#
protected override void BuildFeatures()
{
    var faction = Features.Add(new FactionFeature());
    var stats = Features.Add(new StatsFeature());
    var status = Features.Add(new StatusFeature());
    var life = Features.Add(new LifeFeature());

    Features.Add(new WeaponFeature());
    Features.Add(new AttackEmitterFeature(overlapBufferSize: 96));
    Features.Add(new ActionRunnerFeature());
    Features.Add(new SoulStackFeature());
    Features.Add(new DashGainFeature());
}
```

이렇게 나눈 이유는 다음과 같습니다.
1. 캐릭터가 담당하는 책임을 한 클래스에 몰아넣지 않기 위해
2. 플레이어와 적이 같은 기반 구조를 공유할 수 있게 하기 위해
3. 전투 기능을 조합형으로 확장하기 쉽게 만들기 위해

예를 들어 체력은 `LifeFeature`, 상태이상은 `StatusFeature`, 공격 실행은 `ActionRunnerFeature`, 공격 판정 방출은 `AttackEmitterFeature`가 맡습니다.
캐릭터는 이 기능들을 직접 구현하기보다 조립하고 연결하는 역할에 더 가깝습니다.

### Controller 구조
행동 입력은 `IController`를 통해 분리했습니다.
플레이어는 `PlayerController`, 적은 `EnemyBTRunner`처럼 각기 다른 컨트롤러를 붙이되, `Character` 입장에서는 같은 인터페이스로 취급합니다.

```c#
public interface IController
{
    void Tick(Character character, float dt);
    Weapon BuildDefaultWeapon();
}

public interface ICharacterBinder
{
    void Bind(Character character);
}
```

`OnFeaturesBuilt()`에서 컨트롤러를 바인딩하고, 매 프레임 `Tick()`을 호출합니다.
이 구조 덕분에 캐릭터의 런타임 본체와 입력/AI 로직이 자연스럽게 분리됩니다.

```c#
protected override void OnFeaturesBuilt()
{
    _controller = controllerBehaviour as IController;

    if (controllerBehaviour is ICharacterBinder binder)
        binder.Bind(this);

    move?.Bind(this);
    animBridge?.Bind(this);
}

protected override void Update()
{
    base.Update();
    _controller?.Tick(this, Time.deltaTime);
}
```

## Enemy AI

<p align="center" width="100%">
    <img width="80%" src="/assets/img/portfolio/behavior-tree-1.png" alt="개인 프로젝트의 Behavior Tree 구성 화면">
</p>

AI의 경우에는 기획자가 편하게 수정을 진행할 수 있도록 Visual Scripting 가능하도록 제작했습니다. 
기본적인 뼈대는 UnityBehavior를 사용해서 구현했습니다.

### EnemyBTRunner
`EnemyBTRunner`는 적 캐릭터에 바인딩되는 실제 런타임 컨트롤러입니다.
`Character`와 연결되어 적의 상태, 모듈, 무기, 피격 상태를 관리하고, Unity Behavior Graph가 사용할 `EnemyContext`를 준비합니다.
`EnemyContext`는 Behavior Tree 노드들이 공통으로 참조하는 런타임 컨텍스트 객체 역할을 하도록 제작했습니다.

```c#
public sealed class EnemyBTRunner : MonoBehaviour, IController, ICharacterBinder
{
    private Character _owner = null!;
    private EnemyContext _ctx = null!;
    private BehaviorGraphAgent _agent = null!;

    public EnemyContext Context => _ctx;
}
```

`Bind()` 시점에 `EnemyContext`를 만들고, 적이 사용하는 모듈을 등록합니다.
그리고 순찰 경로, 체력 이벤트, 무기 장착까지 함께 초기화합니다.

```c#
public void Bind(Character character)
{
    _owner = character;
    _ctx = new EnemyContext(character, BuildConfig());

    for (int i = 0; i < moduleDefinitions.Length; i++)
    {
        var module = def.CreateRuntime();
        _ctx.RegisterModule(module);
        module.Bind(_ctx);
    }

    _agent = GetComponent<BehaviorGraphAgent>();
}
```

업데이트 루프에서는 사망, 피격, 기절, 액션 취소, 패시브 모듈 업데이트, 활성 액션 갱신 같은 런타임 처리만 담당합니다.
즉, 실제 의사결정은 노드 쪽으로 넘기고, 컨트롤러는 컨텍스트 유지와 상태 정리 역할을 맡도록 나눴습니다.

### EnemyContext
BT 노드가 필요한 대부분의 정보는 `EnemyContext`에 모아 두었습니다.
타겟 탐색, 거리 계산, 이동 입력 변환, 액션 실행, 쿨다운, 패시브 모듈 접근 등을 모두 `EnemyContext` 경유로 사용합니다.

이렇게 한 이유는 노드가 개별 컴포넌트에 직접 접근하기 시작하면 재사용성이 크게 떨어지기 때문입니다.
노드는 "무엇을 할지"에 집중하고, 실제 게임 오브젝트 접근과 공통 로직은 Context에서 처리하는 방향으로 설계했습니다.

### EnemyBTNodes
실제 Behavior Tree의 내용은 `EnemyBTNodes.cs`에 정의한 Condition / Action 노드들로 구성됩니다.

예를 들어 Condition 노드는 다음과 같은 역할을 합니다.
- 타겟이 유효한지 확인
- 공격 범위 안에 들어왔는지 확인
- HP 비율이 특정 값 이하인지 확인
- 현재 액션 모듈이 실행 중인지 확인
- 현재 상태가 Stunned / Dead인지 확인

```c#
public override bool IsTrue()
{
    _ctx ??= EnemyBTNodeHelper.GetContext(GameObject);
    if (_ctx == null || !_ctx.IsTargetValid(_ctx.Target)) return false;
    return _ctx.DistanceToTarget(_ctx.Target!) <= MaxRange.Value;
}
```

Action 노드는 다음과 같은 역할을 합니다.
- 가장 가까운 적대 타겟 탐색
- 타겟 방향으로 회전
- 타겟을 향해 이동
- 이동 중지
- 일정 시간 대기
- 순찰 포인트 순회

```c#
protected override Status OnStart()
{
    var ctx = EnemyBTNodeHelper.GetContext(GameObject);
    if (ctx == null) return Status.Failure;

    if (ctx.TryAcquireTarget(out var target))
    {
        ctx.Target = target;
        return Status.Success;
    }

    ctx.Target = null;
    return Status.Failure;
}
```

이 구조의 장점은 다음과 같습니다.
1. 노드 단위로 기능이 잘게 쪼개져서 그래프에서 재조합이 쉽다.
2. 타겟 탐색, 이동, 순찰, 공격 조건을 데이터처럼 구성할 수 있다.
3. 적 종류가 달라도 Context와 노드 집합을 공유하면서 일부만 바꿀 수 있다.

즉, 지금의 적 AI는 "컨트롤러가 모든 걸 판단하는 구조"가 아니라 "Context를 공유하는 노드 기반 의사결정 구조"에 가깝습니다.

## Game Boot / Manager

### Game.cs
게임 전체 부트는 `Game`에서 시작합니다.
`Game`은 시작 시 필요한 공통 리소스를 준비하고, 로딩 씬을 거쳐 테이블을 읽은 후 타이틀 씬으로 진입시키는 역할을 합니다.

```c#
private async UniTask InitializeAsync()
{
    ImmortalCanvas.Load();

    await LevelManager.Instance.LoadSceneAsync("LoadingScene");
    //...
    await TableManager.Instance.LoadTable();
    IsTableLoaded = true;
    //...
    await LevelManager.Instance.LoadSceneAsync("TitleScene");
}
```

핵심은 다음과 같습니다.
1. 씬이 바뀌어도 유지되어야 하는 UI를 먼저 준비
2. 로딩 씬 진입
3. 게임에서 사용하는 테이블 로딩
4. 준비 완료 후 실제 시작 씬 진입

이렇게 해두면 캐릭터나 UI가 테이블 준비 여부를 확인하고 안전하게 초기화할 수 있습니다.

### ObjectPoolManager
`ObjectPoolManager`는 풀을 생성하고, 동기/비동기 방식으로 오브젝트를 가져오고, 다시 반환하는 역할을 담당합니다.
UI 팝업, UI 컴포넌트, 이펙트처럼 반복 생성되는 오브젝트를 재사용하기 위한 매니저입니다.

```c#
public GameObject GetObjectSync(string tableName)
{
    if (!_mapObjectPool.ContainsKey(tableName))
        Generate(tableName, false);

    return _mapObjectPool[tableName].GetObjectSync();
}
```

UI 쪽에서는 팝업과 컴포넌트 반환에 사용하고, 캐릭터 쪽에서는 HP Bar나 풀링 오브젝트 반환에 활용합니다.

### TableManager
`TableManager`는 게임에서 사용하는 데이터 컨테이너를 로드하는 역할을 합니다.
`UniTask.WhenAll`로 여러 테이블을 동시에 읽도록 구성했습니다.

```c#
public async UniTask LoadTable(Action onFinished = null)
{
    await UniTask.WhenAll(
        Test.Load(),
        ObjectPool.Load()
    );

    onFinished?.Invoke();
}
```

즉, `Game`은 부트 흐름을 담당하고, `TableManager`는 실제 데이터 준비를 맡는 구조입니다.

### MessageSystem
`MessageSystem`은 프로젝트 전반에서 사용하는 메시지 허브입니다.
point-to-point와 pub/sub을 모두 지원하고, 중요한 특징은 "이번 프레임에 발행된 이벤트를 프레임 끝에서 일괄 처리한다"는 점입니다.

이 방식으로 같은 프레임 내 코드 실행 순서에 따라 이벤트를 듣고 못 듣는 문제가 생기는 것을 줄이려고 했습니다.

```c#
private List<PublishedEvent> publishedOnThisFrame =
    new List<PublishedEvent>();
```

또한 구독/해제 요청도 즉시 수정하지 않고 모아두었다가 한 번에 반영합니다.
이 덕분에 이벤트 콜백 안에서 다시 구독/해제를 건드려도 컬렉션 수정 문제를 줄일 수 있습니다.

## UI 구조

UI는 `UIManager`, `UISceneRoot`, `ImmortalCanvas`를 중심으로 구성했습니다.

### UIWindow / UIPopup 분리
UI는 크게 Window와 Popup으로 나뉩니다.
Window는 화면의 메인이 되는 UI이고, Popup은 그 위에 중첩 가능한 보조 UI입니다.

```c#
public UIWindow CurrentWindow => windowStack.TryPeek(out var currWindow) ? currWindow : null;
public int GetPopupStackCount => popupStack.Count;
```

UI의 생명 주기는 
Window의 경우에는 `Load()`, `Begin()`, `Resume()`, `Pause()`, `Finish()`의 생명주기를 가지고  
Popup의 경우에는 `Begin()`, `OpenAnimation()`, `CloseAnimation()`, `Finish()`의 생명 주기를 가집니다.

### Canvas 보장
UI 쪽에서 중요한 부분 중 하나는 "Canvas가 항상 존재한다고 가정하지 않는다"는 점입니다.
`UIManager`는 윈도우나 팝업을 열기 전에 `EnsureUIRoot()`를 호출해서 `UISceneRoot`가 있는지 확인합니다.

```c#
private bool EnsureUIRoot()
{
    if (windowTrans != null && popupTrans != null)
        return true;

    if (_sceneRoot == null)
        _sceneRoot = Object.FindAnyObjectByType<UISceneRoot>();

    if (_sceneRoot == null)
        _sceneRoot = TryCreateSceneRootFromBundle();

    if (_sceneRoot == null)
        _sceneRoot = CreateRuntimeSceneRoot();
}
```

즉, UI Scene Root가 이미 씬에 있으면 그것을 사용하고, 없으면 번들에서 로드하고, 그것도 없으면 런타임에 직접 생성합니다.
이 덕분에 어떤 씬에서도 Canvas가 없어서 UI가 뜨지 않는 문제가 발생하지 않습니다.

### ImmortalCanvas
`ImmortalCanvas`는 씬 전환과 무관하게 유지되어야 하는 공통 UI를 담당합니다. 
현재는 FadeInOut 연출을 위한 UI, 입력을 막기 위한 UI가 들어가 있습니다.

```c#
public static ImmortalCanvas Load()
{
    if (_instance != null)
        return _instance;

    var canvasPrefab = BundleManager.Instance.LoadBundleSync(ImmortalCanvasBundleName) as GameObject;
    var canvasInstance = Object.Instantiate(canvasPrefab);
    DontDestroyOnLoad(canvasInstance);
}
```

즉, 씬별 UI 루트는 `UISceneRoot`, 전역 유지 UI는 `ImmortalCanvas`로 나눈 구조입니다.

## Tool

Addressable 리소스 관리를 돕기 위한 자동화 에디터 툴을 별도로 제작해 사용하고 있습니다.
리소스를 카테고리 단위로 정리하고, 매핑 작업을 반복 수작업으로 처리하지 않도록 보조하는 용도입니다.

이 부분은 현재 프로젝트 코드의 본질적인 런타임 시스템보다는 작업 효율과 리소스 관리 편의성을 높이기 위한 툴링 영역으로 보고 있습니다.

## 정리

현재 개인 프로젝트 코드에서는 다음 방향을 기준으로 작업했습니다.

- `Character`를 중심으로 Feature와 Controller를 조립하는 구조
- 적 AI는 `EnemyBTRunner`와 `EnemyBTNodes`를 사용하는 BT 중심 구조
- `Game`에서 부트 흐름을 잡고, 데이터/오브젝트풀/메시지 시스템을 각 매니저가 담당하는 구조
- UI는 Canvas 존재를 보장하는 루트 생성 흐름과 Window/Popup 분리 구조
- Addressable 작업은 별도 자동화 툴로 관리

추후에는 실제 플레이 화면과 디버그 캡처를 추가해, 코드 중심 문서에서 한 단계 더 발전시킬 예정입니다.
