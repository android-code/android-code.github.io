---
layout: page
title: Categories
permalink: /categories/
---

{% for category in site.categories %}
  {% include post-category.html %}
{% endfor %}
