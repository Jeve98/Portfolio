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

그리고 Occlusion Culling을 적용한 결과, 저희는 가장 프레임 드랍이 심한 곳을 기준으로 평균 15프레임에서 평균 60프레임으로 약 300% 성능을 향상시킬 수 있었습니다.

### Jitter 완화 작업

특정 증강을 획득했을 때 Jitter가 발생하는 것을 파악하여, 이를 중심으로 원인을 찾았습니다. 저희는 서버와 클라이언트가 서로 다른 시간축으로 상태를 각자 변경하고 있었기 때문에 여기에 네트워크 지연이 심할 경우 Jitter가 발생하는 것으로 판단하였습니다.

이에 상태값을 RPC를 통하여 한번 더 확정 짓는 방어코드를 추가하고 증강이 적용되는 시점을 의도적으로 동기화를 확인한 후로 미룸으로써 Jitter 발생을 안정화할 수 있었습니다.
