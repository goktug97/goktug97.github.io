---
layout: page
title: Demos
permalink: /demos
---

{% for category in site.categories %}
{% if category[0] == "demo" %}
<ul>
{% for post in category[1] %}
<li><a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>
{% endif %}
{% endfor %}
