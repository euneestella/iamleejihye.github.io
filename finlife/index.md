---
layout: default
work: true
main: true
title: Life in FINLAND
description: 핀란드 교환학생 이야기
project-header: true
header-img: "img/project_bg.jpg"
---

<div class="catalogue">
{% assign sorted = site.pages | sort: 'order' | reverse %}
{% for page in sorted %}
{% if page.finlife == true %}

     {% include post-list.html %}

{% endif %}
{% endfor %}
</div>