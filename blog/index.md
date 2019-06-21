---
layout: default
work: true
main: true
title: Blog
description: 소소한 일상을 흘려보내되, 잊지 말아요.
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

