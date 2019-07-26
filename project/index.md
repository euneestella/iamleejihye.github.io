---
layout: default
title: "Project"
description: 지금까지 작업한 프로젝트
main: true
project-header: true
header-img: img/about.jpg
---

<ul class="catalogue">
{% assign sorted = site.pages | sort: 'order' | reverse %}
{% for page in sorted %}
{% if page.blog == true %}
{% include post-list.html %}
{% endif %}
{% endfor %}
</ul>
