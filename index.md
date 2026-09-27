---
title: Azure에서 AI 에이전트 개발
permalink: index.html
layout: home
---

다음 실습은 Microsoft Azure에서 AI 에이전트를 빌드할 때 개발자가 수행하는 일반적인 작업을 직접 해보며 학습할 수 있도록 구성되어 있습니다.

> **참고**: 실습을 완료하려면 필요한 Azure 리소스와 생성형 AI 모델을 프로비저닝할 수 있는 충분한 권한과 할당량이 있는 Azure 구독이 필요합니다. 아직 구독이 없다면 [Azure 계정](https://azure.microsoft.com/free)에 가입할 수 있습니다. 신규 사용자에게는 첫 30일간 크레딧이 제공되는 무료 평가판 옵션이 있습니다.

## 실습

<hr>

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions-kr/Exercises'" | where_exp:"page", "page.lab.duration" | sort: "url" %}
{% for activity in labs  %}

### [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }})

{% if activity.lab.level %}**수준**: {{activity.lab.level}} \| {% endif %}{% if activity.lab.duration %}**소요 시간**: {{activity.lab.duration}}분{% endif %}

*{{activity.lab.description}}*
<hr>
{% endfor %}


> **참고**: 이 실습은 단독으로 완료할 수도 있지만, [Microsoft Learn](https://learn.microsoft.com/training/paths/develop-ai-agents-on-azure/)의 모듈과 함께 학습하도록 설계되었습니다. 해당 모듈에서는 이 실습의 기반이 되는 개념을 더 깊이 있게 다룹니다.
