---
layout: default
title: Planning
permalink: /planning
---

{% for post in site.posts %}
{% if post.category == "Game_Planning" %}

<div class="posts-container">
<h1>
<a href="{{ site.github.url }}{{ post.url }}">{{ post.title }}</a>
</h1>
{% if post.image %}
<div class="thumbnail-container">
<a href="{{ site.github.url }}{{ post.url }}">
<img src="{{ site.github.url }}/assets/img/{{ post.image }}" alt="{{ post.title }}">
</a>
</div>
{% endif %}

      {% if post.period or post.stack %}
        <div class="period">
          {% if post.period %}
            <span class="period-text">{{ post.period }}</span>
          {% endif %}
          {% if post.stack %}
            <span class="title-separator"> | </span>
            <span class="stack-icons">
              {% for tech in post.stack %}
                <img src="{{ site.github.url }}/assets/icons/{{ tech }}.svg" class="stack-icon" style="width: 22px; height: 22px;">
              {% endfor %}
            </span>
          {% endif %}
          {% if post.period %}
            <span class="period-text">{{ post.category }}</span>
          {% endif %}
        </div>
      {% endif %}
      <p class="post">{{ post.summary | newline_to_br }}</p>
      <a class="post" href="{{ site.github.url }}{{ post.url }}">Read More</a>
    </div>

{% endif %}
{% endfor %}
