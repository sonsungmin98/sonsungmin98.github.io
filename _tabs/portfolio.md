---
title: Portfolio
icon: fas fa-briefcase
order: 1
---

# Portfolio

프로젝트와 기술 문서를 정리한 포트폴리오입니다.
게임 클라이언트 개발 경험과 서버 구현 문서를 함께 아카이빙하고 있습니다.

## Project

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="text-align: left; color: #94A3B8; font-weight: 500; padding: 0.75rem 0.5rem;">소속</th>
      <th style="text-align: left; color: #E2E8F0; font-weight: 600; padding: 0.75rem 0.5rem;">Project</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="color: #94A3B8; padding: 0.75rem 0.5rem;">콩스튜디오</td>
      <td style="color: #E2E8F0; font-weight: 600; padding: 0.75rem 0.5rem;">가디언 테일즈</td>
    </tr>
    <tr>
      <td style="color: #94A3B8; padding: 0.75rem 0.5rem;">펄어비스</td>
      <td style="color: #E2E8F0; font-weight: 600; padding: 0.75rem 0.5rem;">검은사막 모바일</td>
    </tr>
    <tr>
      <td style="color: #94A3B8; padding: 0.75rem 0.5rem;">4시 33분</td>
      <td style="color: #E2E8F0; font-weight: 600; padding: 0.75rem 0.5rem;">Meta Bolts</td>
    </tr>
    <tr>
      <td style="color: #94A3B8; padding: 0.75rem 0.5rem;">2022 대학교 졸업 작품</td>
      <td style="color: #E2E8F0; font-weight: 600; padding: 0.75rem 0.5rem;">ToyLand</td>
    </tr>
    <tr>
      <td style="color: #94A3B8; padding: 0.75rem 0.5rem;">2021 대학교 팀 프로젝트</td>
      <td style="color: #E2E8F0; font-weight: 600; padding: 0.75rem 0.5rem;">Lost Home</td>
    </tr>
    <tr>
      <td style="color: #94A3B8; padding: 0.75rem 0.5rem;">2021 대학교 팀 프로젝트</td>
      <td style="color: #E2E8F0; font-weight: 600; padding: 0.75rem 0.5rem;">도깨비</td>
    </tr>
    <tr>
      <td style="color: #94A3B8; padding: 0.75rem 0.5rem;">개인 프로젝트</td>
      <td style="color: #E2E8F0; font-weight: 600; padding: 0.75rem 0.5rem;">개인 프로젝트</td>
    </tr>
  </tbody>
</table>

## Project

### SoulTaker

<p align="left" width="100%">
    <img width="80%" src="/assets/img/portfolio/SoulTaker.png" alt="SoulTaker">
</p>

- Unity 기반 액션 프로젝트
- Character 구조, Behavior Tree 기반 적 AI, Game Boot, UI 프레임워크 중심으로 정리한 프로젝트
- 최근 작성한 실제 코드 기준으로 시스템 구조를 문서화

[SoulTaker 문서 보기]({{ '/posts/SoulTaker/' | relative_url }})

### MMORPG Server

<p align="center">
  <img
    src="https://github.com/user-attachments/assets/4a6f27b1-b185-44d9-a4ef-4da3f36a7eee"
    alt="MMORPG 서버 이동 동기화 예시 이미지"
    style="display: block; width: 100%; max-width: 720px; margin: 0 auto; border-radius: 14px;"
  >
</p>

- C++, IOCP, Protobuf, Unreal 5.4, MSSQL, Python
- MMORPG를 기준으로 개인 제작 중인 서버 프로젝트
- 서버 Core와 콘텐츠 동기화 문서를 정리해두었습니다

{% assign cppserver_posts = site.posts | where_exp: "post", "post.categories contains 'Portfolio' and post.categories contains 'CPPServer'" %}
{% for post in cppserver_posts %}
- [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}

### ToyLand

<p align="left" width="100%">
    <img width="80%" src="/assets/img/portfolio/ToyLand.jpg" alt="MetaBolts">
</p>

[YouTube](https://www.youtube.com/watch?v=ML6XieDvd6Y){: .btn .btn-outline-primary }

- Unreal 4.26.2
- 3D 액션 / 핵 앤 슬래시
- Player Character, UI, Interactive Object 작업 담당
- BIC, 청강 크로니클, 버닝비버 전시

[ToyLand 문서 보기]({{ '/posts/ToyLand/' | relative_url }})

### 협업 툴

<p align="center">
  <img
    src="https://github.com/user-attachments/assets/ef2092f0-9c48-44dd-8130-1a689c44a3ef"
    alt="협업 툴 대표 화면"
    style="display: block; width: 100%; max-width: 720px; margin: 0 auto; border-radius: 14px;"
  >
</p>

- Unity 기반 툴링
- 파일 관리, GitHub Issue 관리 등 협업 생산성을 위한 기능 제작
- 엔진 종속 툴과 외부 툴을 함께 확장 중

[협업툴 문서 보기]({{ '/posts/협업툴/' | relative_url }})

### 도깨비

<p align="left" width="100%">
    <img width="80%" src="/assets/img/portfolio/Dokkaebi.jpg" alt="MetaBolts">
</p>

[YouTube](https://www.youtube.com/watch?v=ubSrKggzOyk){: .btn .btn-outline-primary }

- Unity 2020.2.3.f1
- 2D 액션 플랫포머
- Enemy, UI, 현실 세계 연출, 튜토리얼 제작 담당
- BIC, Unity MWU, BIGS 전시

[도깨비 문서 보기]({{ '/posts/도깨비/' | relative_url }})

## Coding Style

작업의 우선순위는 안정성, 유지보수성, 성능 순으로 두고 있습니다.
코드는 스스로 설명해야 한다고 생각하며, 과한 최적화나 모호한 이름보다 읽기 쉬운 구조와 명확한 추상화를 우선합니다.

- 안정성 우선
- 유지보수성과 확장성 중시
- 모호한 축약어 지양
- `const` 적극 사용
- 완전한 `is-a` 관계가 아니면 `has-a`로 설계
- 의도를 설명하는 주석만 제한적으로 사용
