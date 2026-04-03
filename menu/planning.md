---
layout: default
title: Planning
permalink: /planning
---

<div class="portfolio-grid">
  {% for post in site.posts %}
    {% if post.category == "Game_Planning" %}
      
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
              <span class="period-text">게임 기획</span>
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

          <p class="post" style="height: 4.8em; overflow: hidden; display: -webkit-box; -webkit-line-clamp: 3; -webkit-box-orient: vertical; text-overflow: ellipsis;">
            {{ post.summary | newline_to_br }}
          </p>
          <a class="post" href="{{ site.github.url }}{{ post.url }}">Read More</a>
        </div>
      </div>

    {% endif %}

{% endfor %}

</div>

<style>
.portfolio-grid {
    display: flex;
    flex-wrap: wrap; /* 내용이 많으면 다음 줄로 넘김 */
    gap: 40px;      /* 카드 사이의 간격 */
    align-items: flex-start;
}

.column-item {
    flex: 1 1 calc(50% - 20px); /* 좌우 2개씩 배치 (간격 제외 50%) */
    min-width: 300px;           /* 모바일 등 좁은 화면 대응 */
    box-sizing: border-box;
}

@media (max-width: 850px) {
    .column-item {
        flex: 1 1 100%;          /* 화면이 좁아지면 한 줄에 하나씩 */
    }
}
</style>
