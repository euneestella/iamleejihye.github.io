---
layout: default
work: true
main: true
title: Study
description: Data Science 공부를 기록합니다.
project-header: true
header-img: "img/about.jpg"
---

<div class="catalogue">
{% assign sorted = site.pages | sort: 'order' | reverse %}
{% for page in sorted %}
{% if page.projects == true %}

     {% include post-list.html %}

{% endif %}
{% endfor %}
</div>

