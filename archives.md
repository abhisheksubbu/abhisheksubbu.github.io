---
title: Archives
layout: default
---

{% for post in site.posts %}
{% assign currentdate = post.date | date: "%B %Y" %}
{% if currentdate != date %}
{% unless forloop.first %}</ul>{% endunless %}

<h1 id="y{{post.date | date: "%B %Y"}}">{{ currentdate }}</h1>
<ul>
{% assign date = currentdate %}
{% endif %}
<li><a href="{{ post.url }}">{{ post.title }}</a></li>
{% if forloop.last %}</ul>{% endif %}
{% endfor %}