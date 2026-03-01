---
layout: post
title: "프라가라흐"
author: "전연욱"
categories: Game Development
tags: [Phaser, Web_Game, 2D, RPG]
youtube: q09cp5137Xc
image: fragarach_temp.png
period: 2025-09-30 ~ 2025-12-25
stack: [phaser, django, web]
summary: |
  - Min-Heap 기반 보스 패턴 설계
  - 안전 구역 기반 스폰 시스템 구현
  - 컷씬 중복 재생 방지 시스템 구현
  - Phaser 구조 기반 Scene 로딩 설계
  - API 서버 구축 (Django)
---

### Min-Heap 기반 보스 패턴

보스의 패턴을 랜덤하게 구성하는 방법에는 여러 가지가 있습니다. 예컨대 내장된 랜덤 함수를 사용하거나, 반복문 내부에서 각 패턴의 상태(쿨타임)를 확인하는 것으로 구현할 수 있습니다.
하지만 전자는 강력한/약한 패턴이 무한히 반복되며 플레이어의 운에 따라 편차가 큰 경험, 속칭 억까/억빠가 발생할 수 있으며 후자는 특정 시점에 강력한 패턴이 몰아치는 사이클이 발생할 수도 있습니다.
위와 같은 문제를 해결하기 위해 각 패턴에 우선순위를 부여하고 Min-Heap 자료구조를 이용하여 패턴이 적절하게 섞여 등장할 수 있도록 설계했습니다. Min-Heap은 삽입 시에 Sort 되는 계산이 필요하므로 앞선 방법들에 비해 시간이 더 필요하긴 하지만, 이 역시도 log N이라는 짧은 시간만을 소모하므로 충분히 대처할 수 있습니다. 또한 쿨타임이 끝난 스킬만 삽입하는 것으로 패턴 사용 시에 해당 패턴의 상태(쿨타임)을 확인하는 절차를 제거할 수 있습니다.

초기 구현에는 각 패턴 사용을 switch문으로 분기 처리했습니다. 동작은 문제없었지만, 이러한 설계는 새로운 패턴이 추가될 때마다 switch문을 수정해야 하므로 확장성이 낮고 유지 보수성이 낮은 구조였습니다. 또한 코드 리뷰나 유지보수 시의 가독성 역시 크게 해쳤습니다.
이에 해당 부분의 리팩토링 필요성이 느껴졌고, 이를 패턴 명을 Key, 쿨타임을 Value로 갖는 Dictionary를 Value로 하여 우선순위를 Key로 가지는 패턴 변수를 만들어 해결했습니다.

```
const BossPatterns = {
  coffin: {
    999: { key: 'thunder', cool: 2.5, init: 1 },
    1: { key: 'summons', cool: 90, init: 5 },
    2: { key: 'fireshoot', cool: 30, init: 10 },
    3: { key: 'hassle', cool: 40, init: 30 },
  },
  vampire: {
    999: { key: 'batswarm', cool: 2.5, init: 1 },
    1: { key: 'summons', cool: 65, init: 23 },
    2: { key: 'windAoe', cool: 30, init: 15 },
    3: { key: 'reflectvoid', cool: 30, init: 30 },
  }
};
```

리팩토링을 거친 덕분에 추가 구현된 패턴은 기획 의도에 맞는 우선순위, 패턴 명, 쿨타임만 해당 변수에 추가해도 추가적인 코드 수정 없이 정상적으로 동작시킬 수 있었습니다. 즉, 확장성을 향상할 수 있었습니다. 뿐만 아니라 코드 길이를 효과적으로 줄여 가독성을 높이고 휴먼 에러를 줄일 수 있었습니다.

```
function CastSkill(priority, scene){
    // 보스 이름에 맞는 패턴 테이블 사용
    const patternTable = BossPatterns[BossInstance.name];
    if(!patternTable) return;

    // 우선순위에 해당하는 패턴 데이터
    const pattern = patternTable[priority];
    if(!pattern) return;

    const {key, cool} = pattern;
    BossInstance.patternSet[key].tryCast(scene, BossInstance);  // 패턴 명으로 패턴 사용
    cooltime(scene, priority, cool);  // 쿨타임
}

function initPattern(scene){
    // 보스 이름에 맞는 패턴 테이블 사용
    const patternTable = BossPatterns[BossInstance.name];
    if(!patternTable) return;

    // 패턴 테이블을 순회하며 min-heap에 추가
    for(const priority in patternTable){
        const {cool, init} = patternTable[priority];
        cooltime(scene, Number(priority), init ?? cool);  // 초기화 쿨타임이 없을 경우, 일반 쿨타임 사용
    }
}

export function ChooseNextSkill(scene) {
    // 보스 생성 확인
    if (!BossInstance) return;

    // 보스 패턴 초기화
    if (!BossInstance.isInit) {
        BossInstance.isInit = true;
        initPattern(scene);
        return; // 초기화 직후에는 공격하지 않음
    }

    // 공격 중이면 대기
    // isAttack 값은 각 패턴 코드에서 패턴 사용 종료 후, false로 변환
    if (BossInstance.isAttack) return;

    // 패턴 heap 확인
    if (!BossInstance.nextPattern || BossInstance.nextPattern.size() === 0) return;

    BossInstance.isAttack = true;

    const priority = BossInstance.nextPattern.pop();
    CastSkill(priority, scene);
}

export function cooltime(scene, target, cool) {
    scene.time.delayedCall(cool * 1000, () => {
        BossInstance.nextPattern.push(target);
    })
}
```

### 안전 구역 기반 스폰 시스템

### 컷씬 중복 재생 방지 시스템

### Phaser 구조 기반 Scene 로딩

Phaser의 경우 한 번에 모든 Scene을 로드

## API 서버
