---
title: Portfolio
icon: fas fa-briefcase
order: 1
---

포트폴리오 문서를 원본 디렉토리 구조에 맞춰 정리했습니다.

## Structure
- Portfolio
- Project
- CPPServer
- TechnicalDocs (planned)

## Project
{% assign project_posts = site.posts | where_exp: "post", "post.categories contains 'Portfolio' and post.categories contains 'Project'" %}
{% for post in project_posts %}
- [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}

## CPPServer
{% assign cppserver_posts = site.posts | where_exp: "post", "post.categories contains 'Portfolio' and post.categories contains 'CPPServer'" %}
{% for post in cppserver_posts %}
- [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}

## Notes
- 전체 카테고리 목록은 [Categories]({{ '/categories/' | relative_url }})에서 확인할 수 있습니다.
- 기술문서는 이후 `Portfolio > TechnicalDocs` 구조로 같은 방식으로 확장하면 됩니다.
