---
layout: default
title: Project
permalink: /project
---

<div class="intro-header" style="padding: 0px 0; margin-bottom: 50px;">
  <h2 style="font-size: 2rem; font-weight: 900; letter-spacing: -1px; margin-bottom: 20px;">
    전연욱
  </h2>
  <p style="font-size: 1rem; color: #555; line-height: 1.5; max-width: 900px;">
    <strong>"수치와 근거를 기반으로 플레이어의 추억을 설계합니다."</strong><br>
    단순히 기능을 구현하는 것을 넘어, 왜 그 기술/패턴을 사용해야하는지 선택에 근거를 설명하고 수치로 증명합니다.<br>
    근거가 모여 시스템의 안정적인 서비스를 만들어내고, 이는 수치와 플레이어의 UX로써 완성됩니다.<br>
    <br>
    LOD 대신 적용한 Occlusion Culling을 통해 13fps를 63fps로 끌어올리고,<br>
    GPU Instancing과 오브젝트 풀링으로 드로우콜은 3500에서 180으로, 메모리 점유율은 60mb(추정치)에서 15mb로 감소시켰습니다.<br>
    <br>
    Windows 빌드에서 머무르지 않고 WebGL 같이 또 다른 플랫폼에 도전하며<br>
    그 과정에서 보안 취약점에 대비하고 플랫폼에 맞는 최적화 방법을 찾습니다.
  </p>
  {% include skills-credentials.html %}
</div>

<div class="portfolio-grid">
  {% for post in site.posts %}
    {% if post.category == "Game_Development" %}
      <div class="column-item">
        {% include post-card.html %}
      </div>
    {% endif %}
  {% endfor %}
</div>
