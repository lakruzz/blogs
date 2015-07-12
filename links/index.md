---
layout: default
title: Links
---

<ul>
{% for link in site.data.urls %}
  <li>
    <a href="{{ link.url }}" title="{{link.description }}">
      {{ link.name }}
    </a>

    {% if link.suburls %}
    <ul>
      {% for sublink in link.suburls %}
        <li>
          <a href="{{ sublink.url }}" title="{{sublink.description }}">
            {{ sublink.name }}
          </a>
        </li>
      {% endfor %}
    </ul>
    {% endif %}
  </li>
{% endfor %}
</ul>
