---
layout: default
title: Project
permalink: /project
---

<div class="portfolio-grid">
  {% for post in site.posts %}
    {% if post.category == "Game_Development" or post.category == "Not_Game_Development" %}
      
      <div class="column-item">
        <div class="posts-container" style="margin-bottom: 50px;">
          <h1>
            <a href="{{ site.github.url }}{{ post.url }}">{{ post.title }}</a>
            {% if post.tags and post.tags.size > 0 %}
              <span class="post">
                <span class="title-separator">|</span>
                {{ post.tags | join: ", " }}
              </span>
            {% endif %}
          </h1>

          {% if post.image %}
            <div class="thumbnail-container">
              <a href="{{ site.github.url }}{{ post.url }}">
                <img src="{{ site.github.url }}/assets/img/{{ post.image }}" alt="{{ post.title }}" style="width: 100%; height: 250px; object-fit: cover; border-radius: 8px;">
              </a>
            </div>
          {% endif %}

          {% if post.period or post.stack %}
            <div class="period">
              <span class="period-text">
                {% if post.category == "Game_Development" %}게임 프로젝트{% else %}프로젝트{% endif %}
              </span>
              {% if post.period %}
                <span class="title-separator"> | </span>
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
            </div>
          {% endif %}

          <p class="post" style="height: 4.8em; overflow: hidden; display: -webkit-box; -webkit-line-clamp: 3; -webkit-box-orient: vertical; text-overflow: ellipsis; margin-top: 10px;">
            {{ post.summary | newline_to_br }}
          </p>

          <a class="post" href="{{ site.github.url }}{{ post.url }}" style="font-weight: bold;">Read More →</a>
        </div>
      </div>

    {% endif %}

{% endfor %}

</div>

<style>
.portfolio-grid {
    display: flex;
    flex-wrap: wrap;
    gap: 40px;
    align-items: flex-start;
}

.column-item {
    /* 좌우 2개씩 배치 */
    flex: 1 1 calc(50% - 20px);
    min-width: 320px;
    box-sizing: border-box;
}

/* 모바일 화면 대응 */
@media (max-width: 850px) {
    .column-item {
        flex: 1 1 100%;
    }
}
</style>
