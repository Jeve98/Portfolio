---
layout: default
title: Project
permalink: /project
---

<div class="portfolio-grid">
  {% for post in site.posts %}
    {% if post.category == "Game_Development" %}
      <div class="column-item">
        {% include post-card.html %}
      </div>
    {% endif %}
  {% endfor %}
</div>

<style>
/* 기존 스타일 유지 */
.portfolio-grid { display: flex; flex-wrap: wrap; gap: 40px; align-items: flex-start; }
.column-item { flex: 1 1 calc(50% - 20px); min-width: 300px; box-sizing: border-box; }
@media (max-width: 850px) { .column-item { flex: 1 1 100%; } }
</style>
