---
layout: page
title: Blog Archive
---

{% for tag in site.tags %}
  <!--<h3>{{ tag[0] }}</h3>-->
  <ul>
    {% assign sorted_posts = tag[1] | sort: "date" %}
    {% for post in sorted_posts %}
      <li><a href="{{ post.url }}">{{ post.date | date: "%m-%d" }} : {{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}