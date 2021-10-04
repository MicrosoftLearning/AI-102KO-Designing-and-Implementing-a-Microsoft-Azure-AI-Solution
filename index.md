---
title: 온라인 호스팅 지침
permalink: index.html
layout: home
---

# 콘텐츠 디렉터리

이 리포지토리에는 Microsoft 과정 [AI-102 Microsoft Azure AI 솔루션 설계 및 구현](https://docs.microsoft.com/learn/certifications/courses/ai-102t00) 및 이 과정에 해당하는 [Microsoft Learn의 자기 주도적 모듈](https://aka.ms/AzureLearn_AIEngineer-kor)용 실습 랩 연습이 포함되어 있습니다. 연습은 학습 자료와 함께 진행할 수 있으며, 설명되어 있는 기술을 사용하여 연습할 수 있게 도와줍니다.

이러한 연습을 완료하려면 Microsoft Azure 구독이 필요합니다. 강사가 구독을 제공하지 않은 경우 [https://azure.microsoft.com](https://azure.microsoft.com)에서 등록을 하면 무료 평가판을 받을 수 있습니다.

## 랩

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %}
| 모듈 | 랩 |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

