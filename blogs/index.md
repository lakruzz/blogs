---
layout: blogs
title: Praqma Public Blogs
---
<div>

{% for category in site.categories %}
  {% if category.first == "blogs" %}
    {% for posts in category %}
      {% for post in posts %}
{% if post.title != null %}
        <div>
<h1>{{ post.title }}</h1>
<strong>Author:</strong>{{ post.author }}
  <strong>Date:</strong> {{ post.date | date: "%A, %B %d. %Y"}}<br>
        <br>{{ post.content }}<hr>
        </div>
{% endif %}
      {% endfor %}
    {% endfor %}
  {% endif %}
{% endfor %}

</div>
