---
layout: default
work: true
main: true
title: Finlife
description: 핀란드 교환학생 일기
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
