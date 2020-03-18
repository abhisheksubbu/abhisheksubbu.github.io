---
title: Blog Archives
layout: default
---

<h1>{{page.title}}</h1>
<div style="margin-bottom:2rem;">
    <nav aria-label="breadcrumb">
      <ol class="breadcrumb">
        <li class="breadcrumb-item"><a href="/">Home</a></li>
        <li class="breadcrumb-item active" aria-current="page">{{page.title}}</li>
      </ol>
    </nav>
  </div>

{% for post in site.posts %}
{% assign currentdate = post.date | date: "%B %Y" %}
{% if currentdate != date %}
{% unless forloop.first %}</ul>{% endunless %}

<h3 id="y{{post.date | date: "%B %Y"}}">{{ currentdate }}</h3>
<ul>
{% assign date = currentdate %}
{% endif %}
<li><a href="{{ post.url }}">{{ post.title }}</a></li>
{% if forloop.last %}</ul>{% endif %}
{% endfor %}
