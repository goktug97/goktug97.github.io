---
layout: page
title: Research
permalink: /research/
---

{% for category in site.categories %}
{% if category[0] == "research" %}
<ul>
{% for post in category[1] %}
<li><a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>
{% endif %}
{% endfor %}
