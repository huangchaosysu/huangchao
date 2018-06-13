---
layout: archive
title: "我的博客"
permalink: /blog/
author_profile: true
---

{% for collection in site.collections %}
  <h2 id="{{ label | slugify }}" class="archive__subtitle">{{ collection.label }}</h2>
  {% for post in collection.docs %}
    {% unless collection.output == false or collection.label == "posts" %}
      {% include archive-single.html %}
    {% endunless %}
  {% endfor %}
{% endfor %}