---
layout: page
title: Blog Archive
---

{% for tag in site.tags %}
  {% assign sorted_posts = tag[1] | sort: "date" %}
  {% assign current_month = "" %}
  <ul>
  {% for post in sorted_posts %}
    {% assign month = post.date | date: "%B %Y" %}
    {% if month != current_month %}
      {% unless forloop.first %}</ul>
  <div style="border-top: 1px dashed #ccc; margin: 8px 0;"></div>
  <ul>{% endunless %}
      <!--<li><strong>{{ month }}</strong></li>-->
      {% assign current_month = month %}
    {% endif %}
    <li><a href="{{ post.url }}">{{ post.date | date: "%m-%d" }} : {{ post.title }}</a></li>
  {% endfor %}
  </ul>
{% endfor %}