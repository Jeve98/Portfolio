---
layout: post
title: "프라가라흐"
author: "전연욱"
category: Game_Development
tags: [Web_Game, 2D, RPG]
youtube: ["q09cp5137Xc"]
image: fragarach2.png
period: 2025-09-30 ~ 2025-12-25
members: 2인
stack: [phaser, django, vue, web]
summary: |
  - Min-Heap 기반 보스 패턴 설계
  - 안전 구역 기반 스폰 시스템 구현
  - 컷씬 중복 재생 방지 시스템 구현
  - Phaser 구조 기반 Scene 로딩 설계
  - API 서버 구축 (Django)
  - 바이브 코딩된 비정형 구조 재정비
---

### Min-Heap 기반 보스 패턴

보스의 패턴을 랜덤하게 구성하는 방법에는 여러 가지가 있습니다. 예컨대 내장된 랜덤 함수를 사용하거나, 반복문 내부에서 각 패턴의 상태(쿨타임)를 확인하는 것으로 구현할 수 있습니다.

하지만 전자는 강력한/약한 패턴이 무한히 반복되며 플레이어의 운에 따라 편차가 큰 경험, 속칭 억까/억빠가 발생할 수 있으며 후자는 특정 시점에 강력한 패턴이 몰아치는 사이클이 발생할 수도 있습니다.

<div class="comparison-container" style="display: flex !important; flex-direction: row !important; justify-content: center !important; gap: 10px !important; margin: 30px 0 !important;">
  <div style="width: 48% !important; text-align: center !important;">
    <img src="{{ site.github.url }}/assets/source/fumble/fragarach_pattern.png" 
         style="width: 100% !important; height: auto !important; display: inline-block !important; border-radius: 5px;">
    <p style="font-size: 0.85em; color: #888; margin-top: 5px;">보스 패턴</p>
  </div>
</div>

위와 같은 문제를 해결하기 위해 각 **패턴에 우선순위를 부여하고 Min-Heap 자료구조를 이용하여 패턴이 적절하게 섞여 등장**할 수 있도록 설계했습니다. Min-Heap은 삽입 시에 Sort 되는 계산이 필요하므로 앞선 방법들에 비해 시간이 더 필요하긴 하지만, 이 역시도 log N이라는 짧은 시간만을 소모하므로 충분히 대처할 수 있습니다. 또한 쿨타임이 끝난 스킬만 삽입하는 것으로 패턴 사용 시에 해당 패턴의 상태(쿨타임)을 확인하는 절차를 제거할 수 있습니다.

초기 구현에는 각 패턴 사용을 switch문으로 분기 처리했습니다. 동작은 문제없었지만, 이러한 설계는 새로운 패턴이 추가될 때마다 switch문을 수정해야 하므로 확장성이 낮고 유지 보수성이 낮은 구조였습니다. 또한 코드 리뷰나 유지보수 시의 가독성 역시 크게 해쳤습니다.

이에 해당 부분의 리팩토링 필요성이 느껴졌고, 이를 패턴 명을 Key, 쿨타임을 Value로 갖는 Dictionary를 Value로 하여 우선순위를 Key로 가지는 패턴 변수를 만들어 해결했습니다.

```javascript
const BossPatterns = {
  coffin: {
    999: { key: "thunder", cool: 2.5, init: 1 },
    1: { key: "summons", cool: 90, init: 5 },
    2: { key: "fireshoot", cool: 30, init: 10 },
    3: { key: "hassle", cool: 40, init: 30 },
  },
  vampire: {
    999: { key: "batswarm", cool: 2.5, init: 1 },
    1: { key: "summons", cool: 65, init: 23 },
    2: { key: "windAoe", cool: 30, init: 15 },
    3: { key: "reflectvoid", cool: 30, init: 30 },
  },
};
```

리팩토링을 거친 덕분에 새로운 패턴의 추가는 기획 의도에 맞게 우선순위, 패턴 명, 쿨타임만 해당 변수에 추가하는 것으로 충분해졌습니다. 즉, **추상화를 통해 직관적인 패턴의 추가/제거**가 가능해졌으며 개발자가 한번에 확인해야 할 코드 양을 줄이는 것으로 가독성을 높이고 휴먼 에러를 줄일 수 있었습니다.

```javascript
function CastSkill(priority, scene) {
  // 보스 이름에 맞는 패턴 테이블 사용
  const patternTable = BossPatterns[BossInstance.name];
  if (!patternTable) return;

  // 우선순위에 해당하는 패턴 데이터
  const pattern = patternTable[priority];
  if (!pattern) return;

  const { key, cool } = pattern;
  BossInstance.patternSet[key].tryCast(scene, BossInstance); // 패턴 명으로 패턴 사용
  cooltime(scene, priority, cool); // 쿨타임
}

function initPattern(scene) {
  // 보스 이름에 맞는 패턴 테이블 사용
  const patternTable = BossPatterns[BossInstance.name];
  if (!patternTable) return;

  // 패턴 테이블을 순회하며 min-heap에 추가
  for (const priority in patternTable) {
    const { cool, init } = patternTable[priority];
    cooltime(scene, Number(priority), init ?? cool); // 초기화 쿨타임이 없을 경우, 일반 쿨타임 사용
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
  if (!BossInstance.nextPattern || BossInstance.nextPattern.size() === 0)
    return;

  BossInstance.isAttack = true;

  const priority = BossInstance.nextPattern.pop();
  CastSkill(priority, scene);
}

export function cooltime(scene, target, cool) {
  scene.time.delayedCall(cool * 1000, () => {
    BossInstance.nextPattern.push(target);
  });
}
```

### 안전 구역 기반 스폰 시스템

<div class="comparison-container" style="display: flex !important; flex-direction: row !important; justify-content: center !important; gap: 10px !important; margin: 30px 0 !important;">
  <div style="width: 48% !important; text-align: center !important;">
    <img src="{{ site.github.url }}/assets/source/fumble/fragarach_safe_spawn.png" 
         style="width: 100% !important; height: auto !important; display: inline-block !important; border-radius: 5px;">
    <p style="font-size: 0.85em; color: #888; margin-top: 5px;">안전 배치</p>
  </div>
</div>

Phaser 엔진으로 2D 게임을 개발할 경우, 두 오브젝트를 서로 충돌하도록 설정해두어도 겹쳐져서 스폰되면 이들을 분리할 수가 없습니다. 즉, 지형지물 위에 몬스터가 스폰될 경우 그 지형지물 위를 몬스터가 자유롭게 돌아다니는 문제가 발생했습니다.

처음에는 이를 해결하기 위해 지형지물의 collider에서 겹침 판정이 발생할 경우, 해당 몬스터의 위치를 중심으로 가장 가까운 빈 공간으로 밀어내는 방식을 사용했습니다. 하지만 해당 방식은 매 프레임 단위로 아주 조금씩 몬스터를 밀어냈기에 시간이 오래 걸렸습니다. 이를 가속화하고자 빠져나오는 거리를 증가시키면 지형지물을 빠져나온 몬스터가 다른 몬스터에 겹치는 문제가 발생하며 또 다시 불필요한 충돌 연산을 실시하기도 했습니다.

그래서 선택한 방식이 안전 구역 기반의 스폰 시스템이었습니다. 최초 스폰은 맵의 랜덤한 위치에 소환을 하되, 다른 collider와 충돌이 발생할 경우 해당 몬스터를 미리 설정해둔 안전 좌표로 이동시키는 방식으로 몬스터를 지형지물에서 빼내기 위해 매 프레임마다 무거운 충돌 연산을 할 필요가 없었고 빠르게 스폰 위치를 재설정 할 수도 있었습니다. 또한 다수의 안전 좌표를 설정해두는 것으로 혹시나 다수의 몬스터가 겹쳐지는 일을 최소화하려 했고 만약 몬스터 간에 겹쳐짐이 발생하면 둘의 좌표를 동일한 수치만큼 반대 방향으로 밀어내는 로직을 추가하여 대비 하였습니다.

해당 시스템을 적용한 이후, 지형지물 위에 스폰되어 공격할 수 없는 몬스터의 스폰을 막을 수 있었으며 맵 곳곳에 자연스럽게 몬스터를 배치할 수 있었습니다.

### 컷씬 중복 재생 방지 시스템

<div class="comparison-container" style="display: flex !important; flex-direction: row !important; justify-content: center !important; gap: 10px !important; margin: 30px 0 !important;">
  <div style="width: 48% !important; text-align: center !important;">
    <img src="{{ site.github.url }}/assets/source/fumble/fragarach_cutscene.png" 
         style="width: 100% !important; height: auto !important; display: inline-block !important; border-radius: 5px;">
    <p style="font-size: 0.85em; color: #888; margin-top: 5px;">컷씬</p>
  </div>
</div>

특정 scene에서 스토리 진행을 위한 컷씬을 재생하고 있습니다. 다만 해당 scene에서 컷씬을 보았는지에 대해서 확인하는 로직이 없어, 이미 본 컷씬이더라도 해당 scene에 재진입하면 처음부터 다시 봐야하는 문제가 발생했습니다.

컷씬 시청 여부를 DB에 저장하여 해당 문제를 해결하였는데, 이때 중요한 것은 DB 리소스 낭비가 없도록 저장하는 데이터를 최적화하는 것이었습니다. 만약 컷씬이 있는 scene마다 개별 필드를 만든다면 새로운 컷씬이 있는 scene이 추가될 때마다 DB에 새로운 데이터를 삽입해야하며 플레이하는 유저가 많아질 수록 DB에 많은 데이터가 쌓이게 됩니다. 이를 방지하기 위해 유저 테이블에 cutScene 필드를 추가하고 **scene의 넘버를 기반으로 비트 연산**을 적용하는 방법을 사용했습니다. 이를 통해 각 컷씬의 시청 여부를 배타적으로 확인할 수 있었으며 실제로 저장되는 값은 int 값이기에 리소스도 크게 절약할 수 있었습니다. 또한 빠른 속도의 비트 연산을 사용했기 때문에 컷씬 시청 여부 확인을 빠르고 효율적으로 할 수 있었습니다.

```javascript
if ((this.playerStats.cutScene & (1 << this.sceneNumber)) == 0) {
  this.cutscene.play(introScript);
  this.playerStats.cutScene += 1 << this.sceneNumber;
} else {
  this.cutsceneLock = false;
}
```

### Phaser 구조 기반 Scene 로딩

Phaser의 경우, 게임 객체를 생성할 때 모든 Scene을 한번에 로드합니다. 이로인해 각 scene에만 있어야 하는 오브젝트들이 유령 상태로 남아있는 버그가 발생하거나 import 해오는 key가 중복되며 워닝이 발생하곤 했습니다. 이를 해결하기 위해 게임 객체 생성 시에는 로드 씬만을 포함시켜 게임 동작 시에 필요한 데이터를 import 하는 것으로 워닝 발생을 방지하였고 각 scene의 오브젝트 중복 생성을 막았습니다.

### API 서버

삼성 청년 SW·AI 아카데미의 명세에 따라 SQLite3를 사용하여 Rest-API 서버를 구축했습니다. 하지만 프로젝트 진행 중 SQLite3가 가지는 write-lock 기반의 동시성 한계를 인지했습니다. 기존의 구조를 수정하는 것도 한 방법이었지만, 그전에 저희 게임에서 DB에 빈번한 쓰기 작업이 필요한 지를 먼저 분석했습니다. 다행히 1인용 웹 게임이었기에 실제 플레이 시 대부분의 요청은 데이터 조회였고 데이터 쓰기 요청은 회원가입과 저장 시점에서만 발생했습니다. 뿐만 아니라 배포 및 테스트를 해줄 인원이 삼성 청년 SW·AI 아카데미 서울 캠퍼스로 한정되어 있었기 때문에, 모든 이들이 한번에 회원가입 혹은 게임을 저장해도 1000여건이라는 비교적 처리 가능한 수준의 데이터 쓰기 요청이 최대였기 때문에 SQLite3로도 충분히 대응이 가능하다고 생각했습니다.

### 바이브 코딩된 비정형 구조 재정비

함께 프로젝트를 진행했던 팀원은 바이브 코더였기 때문에 각종 기능을 AI를 활용하여 개발했습니다. 이 과정에서 **명확한 컨벤션과 컨텍스트가 정해져있지 않았기에** 코드의 coupling이 굉장히 높은 상황이었습니다. 또한 AI가 생성하는 막대한 양의 코드로 인해 버그가 발생하더라도 그 원인을 분석하기가 굉장히 힘들었습니다. 하지만 해당 사태를 벗어나고자 다시금 AI에 의존하면 더 큰 문제가 추후에 발생할 것이 자명했습니다. 이에 전체 **프로젝트 구조를 분석**하기 시작했습니다. 또한 이를 기반으로 팀원이 구현한 코드가 어떤 로직으로 동작하는지 주석을 포함하여 정리하였습니다.

#### <span style="font-size: 1.1em;">CASE 1. 스킬 시스템 로직</span>

- **버그 상황**  
  스킬 포인트 투자 시, 특정 스킬에만 변경 사항이 적용됨.
- **버그 분석**  
  일부 스킬의 경우 Scene에서 로직이 바인딩 되어 로직이 적용되지 않음을 확인함. 또한 추상 클래스를 상속 받아 구현된 스킬 중 일부에서 가상 함수를 Override하는 과정에서 base가 누락되어 로직이 적용되지 않음을 확인함.
- **해결 방법**  
  팀원이 스스로 문제를 해결할 수 있도록 현재 구현된 각 스킬의 흐름도를 작성하고, 스킬이라는 시스템의 이상적인 구조도와 주석을 포함한 코드를 문서화하여 리팩토링 방향성을 팀원에게 제공함.

#### <span style="font-size: 1.1em;">CASE 2. 데이터 동기화</span>

- **버그 상황**  
  스킬 획득 메커니즘 변경(레벨 도달 → 레벨 도달 및 선행 스킬 해금) 후, 맵을 넘어가거나 재로그인을 했을 때 해금한 스킬이 미해금 상태로 되돌아가는 일이 발생함. 또한 해당 스킬을 스킬 슬롯에 등록한 경우, UI에서는 미해금 상태로 보이며 상호작용이 되지 않았으나 스킬 사용이 가능한 버그가 발견됨.
- **버그 분석**  
  막대한 양의 코드로 인해 모든 로그를 확인하는 것이 불가능하여 프로젝트 전체에서 관련된 함수 및 변수를 검색하여 흐름을 역행하며 분석을 진행함. Vue를 통해 Phaser 엔진을 로드할 때, API로 전송 받은 캐릭터 데이터가 없을 경우를 대비한 레벨 1의 더미 계정이 API 데이터로 덮어씌워져야 하나, 레벨에서만 해당 부분이 누락됨을 확인함. 이로 인해 UI 및 상호작용을 담당하는 Vue에서는 레벨 도달 조건이 해소되지 않은 것으로 인지되어 버그가 발생한 것으로 판단함.
- **해결 방법**  
  코드 흐름은 비교적 단순하였으나 막대한 양의 코드로 인해 디버깅이 불가했으므로, 특정 기능 사용에서 활용되는 함수 및 변수를 정리하여 이후 버그 발생 시 의심되는 부분을 누락없이 빠르게 확인할 수 있도록 팀원과 공유함.
