---
layout: post
title: "마네킹: 헌티드 몰"
author: "전연욱"
category: Game_Development
tags: [3D, 비대칭 PVP]
youtube:
image: mannequin.png
period: 2026-02-20 ~ 2026-03-25
stack: [unity, photon]
summary: |
  - 살인마 진영 기능 구현
  - 상호작용 시스템 구현
  - 증강 시스템 구현
  - Occlusion Culling을 통한 최적화
  - Jitter 완화 작업
---

### 살인마 진영 기능 구현

### 상호작용 시스템 구현

### 증강 시스템 구현

### Occlusion Culling을 통한 최적화

유니티는 3D 프로젝트에 기본적으로 Frustum Culling을 적용하여 시야 밖의 오브젝트를 렌더링하지 않도록 최적화를 진행합니다. 하지만 Frustum Culling의 경우, 맵의 깊이가 깊어지거나 vertex가 복잡한 오브젝트가 시야 내에 있다면 그 성능이 급격하게 떨어지게 됩니다.

해당 프로젝트에서는 AI를 이용하여 구현한 3D mesh가 많았기에 vertex가 복잡한 오브젝트가 많았고, 맵의 크기도 상당하기에 Frustum Culling만으로는 충분한 최적화가 이루어지지 않았습니다.

반면 Occlusion culling의 경우, 특정 오브젝트가 다른 오브젝트에 의해 가려지면 렌더링에서 제외하는 방식이기 때문에 맵의 깊이가 깊어질 수록, 그리고 vertex가 복잡한 오브젝트가 다른 오브젝트에 많이 가려질 수록 효용성이 높아집니다.

해당 프로젝트에서는 오브젝트 간에 가려지는 경우가 많았고 vertex가 복잡한 오브젝트의 수도 많았기 때문에 개별적으로 LOD를 적용하는 것보다 ROI를 높게 가져갈 수 있으면서 더 높은 최적화 효율을 가져올 수 있을 것이라 생각되었습니다.

<div class="comparison-container" style="display: flex !important; flex-direction: row !important; justify-content: center !important; gap: 10px !important; margin: 30px 0 !important;">
  <div style="width: 48% !important; text-align: center !important;">
    <img src="{{ site.github.url }}/assets/source/mannequin_Occlusion_Before.png" 
         style="width: 100% !important; height: auto !important; display: inline-block !important; border-radius: 5px;">
    <p style="font-size: 0.85em; color: #888; margin-top: 5px;">[그림 1] 적용 전 (15fps)</p>
  </div>
  <div style="width: 48% !important; text-align: center !important;">
    <img src="{{ site.github.url }}/assets/source/mannequin_Occlusion_After.png" 
         style="width: 100% !important; height: auto !important; display: inline-block !important; border-radius: 5px;">
    <p style="font-size: 0.85em; color: #888; margin-top: 5px;">[그림 2] 적용 후 (60fps)</p>
  </div>
</div>
<!-- <div style="text-align: center; margin: 40px 0;">
  <div style="display: flex; justify-content: center; gap: 20px; align-items: flex-start; flex-wrap: wrap;">
    <div style="flex: 1; min-width: 300px; max-width: 450px;">
      <img
        src="{{ site.github.url }}/assets/source/mannequin_Occlusion_Before.png"
        alt="Occlusion Culling 적용 전"
        style="width: 100%; border-radius: 5px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"
      >
      <p style="font-size: 0.9em; color: #888; margin-top: 10px;">[그림 1] 적용 전 평균 프레임 (15fps)</p>
    </div>
    <div style="flex: 1; min-width: 300px; max-width: 450px;">
      <img
        src="{{ site.github.url }}/assets/source/mannequin_Occlusion_After.png"
        alt="Occlusion Culling 적용 후"
        style="width: 100%; border-radius: 5px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);"
      >
      <p style="font-size: 0.9em; color: #888; margin-top: 10px;">[그림 2] 적용 후 평균 프레임 (60fps)</p>
    </div>
  </div>
</div> -->

그리고 Occlusion Culling을 적용한 결과, 저희는 가장 프레임 드랍이 심한 곳을 기준으로 평균 15프레임에서 평균 60프레임으로 약 300% 성능을 향상시킬 수 있었습니다.

### Jitter 완화 작업
