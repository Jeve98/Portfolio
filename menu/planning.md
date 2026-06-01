---
layout: default
title: Planning
permalink: /planning
---

<div class="intro-header" style="padding: 0px 0; margin-bottom: 50px;">
  <h2 style="font-size: 2rem; font-weight: 900; letter-spacing: -1px; margin-bottom: 20px;">
    전연욱
  </h2>
  <p style="font-size: 1rem; color: #555; line-height: 1.5; max-width: 1000px;">
    <strong>"트렌드를 읽고 시스템을 짜며, 플레이어의 추억을 만듭니다."</strong><br>
    장르 트렌드 분석을 기반으로 시스템을 설계하고 컨텐츠를 기획합니다.<br>
    <br>
    코로나 이후 게이머의 행동 변화, 리그 오브 레전드의 증강 칼바람 나락 유행, 던전 앤 파이터의 '던닝' 대규모 유입 등<br>
    시장의 흐름을 읽어 기획을 시작하고, 개발 경험을 바탕으로 실현 가능한 기획을 다듬습니다.<br>
    <br>
    초기 컨셉 기획을 실제 개발 프로젝트로 진행한만큼, 기획이 얼마나 프로젝트에 영향을 끼치는지 이해하며<br>
    실현 가능성과 재미 사이의 균형을 찾습니다.
  </p>
  {% include skills-credentials.html %}
</div>

<div class="portfolio-grid">
  {% for post in site.posts %}
    {% if post.category == "Game_Planning" %}
      <div class="column-item">
        {% include post-card.html %}
      </div>
    {% endif %}
  {% endfor %}
</div>
