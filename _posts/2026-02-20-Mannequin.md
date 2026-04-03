---
layout: post
title: "마네킹: 헌티드 몰"
author: "전연욱"
category: Game_Development
tags: [3D, 비대칭 PVP]
youtube: ["AoUDpxRisNE", "1wvWEE_x3T8"]
image: mannequin1.png
period: 2026-02-20 ~ 2026-03-25
members: 6인
stack: [unity, photon]
summary: |
  - 기획
  - 살인마 진영 기능 구현
  - 스폰 시스템 최적화
  - 상호작용 시스템 구현
  - 증강 시스템 구현
  - Occlusion Culling을 통한 최적화
  - Jitter 완화 작업
---

### 기획

대한민국 게임 산업은 날로 성장하고 있습니다. 작년 무역 수출액의 큰 축을 담당하기도 했고, 국가가 나서서 주요 컨텐츠 사업의 일환으로써 성장시키려 하기도 합니다. 하지만 현업자 분들과 얘기해보면 현실은 조금 다릅니다. 이미 자리를 잡고 있는 거대 게임사들 외에는 OTT, 쇼츠, 릴스와 같은 수많은 컨텐츠로 인해 그 자리가 위험하다고 말합니다.

즉발적인 피드백이 가능한 컨텐츠가 많은 만큼, 유저의 평균적인 플레이 타임 및 게임에 원하는 플레이 타임은 크게 감소하고 있습니다. 일례로 리그 오브 레전드(이하 롤)의 협곡 모드는 꾸준히 평균 플레이 타임이 감소해왔으며 플레이 타임이 반토막 난 칼바람 나락 모드에 그 시간을 더욱 가속화시킨 증강이 추가되어 유행하고 있습니다.

얼마 전까지도 큰 인기를 구가하던 로그라이크류 게임은 이러한 트렌드를 굉장히 잘 반영한 장르였습니다. 짧은 **단 판 플레이**에, 랜덤성에 기반하여 매 **플레이마다 새로운 경험**을 제공하니 자연스럽게 **반복 플레이**를 유도할 수 있었습니다.

해당 프로젝트에서는 협력과 배신 그리고 심리전이라는 거대한 기둥을 가진 비대칭 PVP 장르에 이러한 트렌드를 반영하여 **증강 시스템을 통해 플레이 템포를 가속화**하고자 하였습니다.

또한 개발에서 **AI 활용을 적극 권장**하되, 단순히 '만들어줘'로 끝나는 것이 아니라 명확히 어떤 근거에 의거하여 어떤 방식으로 동작하고 **다른 코드와의 정합성과 연동성을 고려하여 프롬프팅하고 개발하도록** 규칙을 정했습니다. 이는 짧은 프로젝트 기간에 맞춰 높은 생산성을 만들어냄과 동시에 AI 사용으로 인한 각종 문제를 효과적으로 대처하기 위함이었습니다.

### 살인마 진영 기능 구현

기본적인 이동 및 1인칭 시점 변경 등 조작을 구현하고 4가지 스킬을 구현하였습니다. (더미 설치, 전이, 교란, 위장)

<div style="display: flex; gap: 30px; margin-bottom: 40px; align-items: flex-start;">
  <div style="flex: 1.2; min-width: 0;">
    <div style="line-height: 0; font-size: 0; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.15); overflow: hidden;">
      <img src="{{ site.github.url }}/assets/source/mannequin/install.gif" style="width: 100%; height: auto; display: block; margin: 0;">
    </div>
  </div>
  <div style="flex: 0.8;">
    <p style="font-size: 0.95rem; color: #666; line-height: 1.6;">
      더미 설치 스킬의 경우, 전이, 교란에 사용되는 더미 마네킹을 맵에 설치하는 스킬로 정해진 구역에만 설치가 가능하도록 설계하였습니다. 일정 거리 내에서만 설치가 가능해야하므로 설치 가능한 구역을 리스트로 보유하였다가 스킬 사용 시에 이를 순회하며 가장 가깝고 설치 가능한 거리 미만에 있는 장소만 특정하게 하였습니다.
    </p>
  </div>
</div>

<div style="display: flex; gap: 30px; margin-bottom: 40px; align-items: flex-start; flex-direction: row-reverse;">
  <div style="flex: 1.2; min-width: 0;">
    <div style="line-height: 0; font-size: 0; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.15); overflow: hidden;">
      <img src="{{ site.github.url }}/assets/source/mannequin/transition.gif" style="width: 100%; height: auto; display: block; margin: 0;">
    </div>
  </div>
  <div style="flex: 0.8;">
    <p style="font-size: 0.95rem; color: #666; line-height: 1.6;">
      전이 스킬의 경우, 더미 마네킹만을 렌더링하는 카메라를 추가하고 이를 기본 시야 위로 렌더링 되도록 설정하여 더미 마네킹을 투시하듯 보는 시점을 구현하였습니다. 이 과정에서 맵에 짙게 추가된 안개에 의해 시야가 가려지는 것을 막기 위해 쉐이더를 제작하여 적용하였습니다.
    </p>
  </div>
</div>

<div style="display: flex; gap: 30px; margin-bottom: 40px; align-items: flex-start;">
  <div style="flex: 1.2; min-width: 0;">
    <div style="line-height: 0; font-size: 0; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.15); overflow: hidden;">
      <img src="{{ site.github.url }}/assets/source/mannequin/derangement.gif" style="width: 100%; height: auto; display: block; margin: 0;">
    </div>
  </div>
  <div style="flex: 0.8;">
    <p style="font-size: 0.95rem; color: #666; line-height: 1.6;">
      교란 스킬은 전이 스킬 사용 시 출력되는 애니메이션을 보고 생존자 플레이어가 쉽게 대처할 수 있던 문제를 심리전으로 확장시키기 위해 추가하였습니다. 모든 더미 마네킹에 애니메이션을 동작시켜 지금 보이는 마네킹이 전이 스킬을 사용한 것인지 교란 스킬을 사용한 것인지 구분할 수 없도록 하였습니다.
    </p>
  </div>
</div>

<div style="display: flex; gap: 30px; margin-bottom: 40px; align-items: flex-start; flex-direction: row-reverse;">
  <div style="flex: 1.2; min-width: 0;">
    <div style="line-height: 0; font-size: 0; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.15); overflow: hidden;">
      <img src="{{ site.github.url }}/assets/source/mannequin/camouflage.gif" style="width: 100%; height: auto; display: block; margin: 0;">
    </div>
  </div>
  <div style="flex: 0.8;">
    <p style="font-size: 0.95rem; color: #666; line-height: 1.6;">
      위장 스킬은 1인칭으로 진행되는 살인마의 시점 상, 다소 부족한 색적 능력을 보완할 수 있도록 3인칭 시점으로 주변을 돌아볼 수 있는 기능을 추가하였으며 인근 거리 내에 살인마가 있으면 생존자에게 재생되는 SE를 재생되지 않게 하는 기능을 가지고 있어, 특정 위치에서 더미 마네킹인 척 위장하며 생존자를 유인할 수 있는 스킬로 설계하였습니다.
    </p>
  </div>
</div>

### 스폰 시스템 최적화

멀티 플레이 환경에서 **네트워크 트래픽을 최소화**하기 위해 살인마 및 생존자의 스폰 데이터를 최적화할 필요가 있었습니다.

기존에는 각 플레이어의 물리적 위치인 transform.position의 세 값(x, y, z)와 해당 위치에서 플레이어가 바라보는 transform.rotation의 세 값(x, y, z)를 각각 전송하여 스폰하였습니다. 하지만 position.y, rotation.x, rotation.z의 경우 고정값으로 전송되었으므로, 이를 제외하고 실제 변경되는 값을 하나로 묶어 전송되도록 데이터 구조를 최적화하여 스폰 시에 필요한 네트워크 트래픽을 절반으로 감소할 수 있었습니다.

```csharp
public (Vector3 position, Quaternion rotation) GetKillerSpawnData()
{
  Vector3 data = killerSpawnPoints[SelectedSpawnIndex];
  Vector3 pos = new Vector3(data.x, 108f, data.z);
  Quaternion rot = Quaternion.Euler(0, data.y, 0);
  return (pos, rot);
}

public (Vector3 position, Quaternion rotation) GetSurvivorSpawnData(int spawnOrder)
{
  int index = (SurvivorStartOffset + spawnOrder) % 5;       // 최대 5인의 생존자
  Vector3 data = survivorSpawnPoints[SelectedSpawnIndex][index];

  Vector3 pos = new Vector3(data.x, 1f, data.z);
  Quaternion rot = Quaternion.Euler(0, data.y, 0);
  return (pos, rot);
}
```

```csharp
private int _survivorSpawnCount = 0;

public void PlayerJoined(PlayerRef player)
{
  if (!Runner.IsServer) return;
  if (SpawnManager.Instance == null || SpawnManager.Instance.SelectedSpawnIndex == -1) return;

  Vector3 spawnPos;
  Quaternion spawnRot;
  NetworkObject prefabToSpawn;
  if (player == Runner.LocalPlayer)
  {
    var data = SpawnManager.Instance.GetKillerSpawnData();
    spawnPos = data.position;
    spawnRot = data.rotation;
    prefabToSpawn = killerPrefab;
  }
  else
  {
    var data = SpawnManager.Instance.GetSurvivorSpawnData(_survivorSpawnCount);
    spawnPos = data.position;
    spawnRot = data.rotation;
    _survivorSpawnCount++;
    prefabToSpawn = survivorPrefab;
  }

  Runner.Spawn(prefabToSpawn, spawnPos, spawnRot, player);
}
```

### 상호작용 시스템 구현

상호작용 시스템을 구현할 때는 **인터페이스**를 기본으로 하여 각 오브젝트에서 기능을 확장할 수 있도록 구성하였습니다.

```csharp
public interface IInteractable
{
    string GetInteractionText(GameObject interactor);   // UI에 표시될 텍스트
    string GetAnimationTrigger();                       // 애니메이션 재생을 위한 트리거 메세지 반환
    void OnInteract(GameObject gameObject);             // 상호작용 실행 시 호출
}
```

상호작용은 크게 파괴, 정화, 뛰어넘기로 나뉘는데 앞선 두 기능은 네트워크 객체로 처리되어야하지만 뛰어넘기는 굳이 네트워크 객체로 다룰 필요가 없었습니다.

즉, 각 기능마다 상속해야하는 클래스가 달랐고(NetworkBehaviour, MonoBehaviour) C#에서는 단일 상속만이 가능하기 때문에 틀을 추상 클래스로 생성하기 보다 인터페이스로 생성하는 것이 효율적이라고 생각하였습니다. 또한 부모/자식의 관계가 수직적으로 확장되는 것이 아니라 '상호작용'이라는 큰 틀에서 서로 다른 기능으로 수평적으로 확장되기 때문에 인터페이스 더욱 적합하다고 판단하였습니다.

```csharp
public class VaultableObject : MonoBehaviour, IInteractable
{
    [SerializeField] private Transform pointA;
    [SerializeField] private Transform pointB;

    public string GetInteractionText(GameObject interactor) => GameplayPromptUtility.Format("G", "담 넘기");

    public string GetAnimationTrigger() => "JumpOver";

    public Vector3 GetVaultExitPosition(Vector3 playerPos)
    {
        float distA = Vector3.Distance(playerPos, pointA.position);
        float distB = Vector3.Distance(playerPos, pointB.position);
        return distA > distB ? pointA.position : pointB.position;
    }

    public void OnInteract(GameObject gameObject)
    {
        KillerController killer = gameObject.GetComponent<KillerController>();
        if (killer != null)
        {
            Vector3 exit = GetVaultExitPosition(killer.transform.position);
            killer.ExecuteVault(exit, GetAnimationTrigger());
        }
    }
}
```

### 증강 시스템 구현

상호작용 때와는 달리, 증강의 경우 호스트/클라이언트 모두가 해당 증강의 활성화 유무를 알아야하므로 NetworkBehaviour를 가질 필요가 있습니다. 또한 '증강'이라는 큰 틀이 유지되며 확장되기에 **추상 클래스**로 구성하는 것이 좋다고 판단하였습니다.

```csharp
public abstract class AbilityBase : NetworkBehaviour
{
    public string abilityName;
    [TextArea] public string description;
    public Sprite icon;

    // 실질적인 동작은 필수이므로 추상 함수로 생성
    public abstract void Activate();

    // 기본적인 동작은 동일하지만(비활성화), Awake 시 추가적인 동작이 필요하다면 수정할 수 있도록 가상 함수로 생성
    protected virtual void Awake()
    {
        this.enabled = false;     // 프리팹에서 활성화 된 상태였다면 비활성화 (방어 코드)
    }
}
```

새로운 증강을 추가함에 있어, 실질적인 동작을 누락하는 경우는 매우 치명적이므로 시스템적으로 이를 방지하고자 Activate 함수는 추상 함수로 생성하였습니다. 반면 Awake 함수의 경우, 기본적으로 프리팹에 추가되는 증강 스크립트가 휴먼 에러로 인해 활성화 상태로 추가된 경우에 방어 코드로써 사용되므로 특수한 경우를 제외하고는 이를 수정할 필요가 없었습니다. 이에 Awake 함수는 가상 함수로 생성하여 필요한 경우에는 쉽게 오버라이드할 수 있게 구현하였습니다.

```csharp
public class SpeedUpAbility : AbilityBase
{
    private ExorcistController _exorcist;
    [SerializeField] private float boostAmount = 150f;    // 증가 속도
    [SerializeField] private float duration = 180f;       // 지속 시간

    public override void Activate()
    {
        _exorcist = GetComponent<ExorcistController>();
        if (_exorcist != null)
        {
            _exorcist.runSpeed += boostAmount;
            Invoke(nameof(Deactivate), duration);
        }
    }

    private void Deactivate()
    {
        if (_exorcist != null) _exorcist.runSpeed -= boostAmount;
        this.enabled = false;
    }
}
```

### Occlusion Culling을 통한 최적화

유니티는 3D 프로젝트에 기본적으로 Frustum Culling을 적용하여 시야 밖의 오브젝트를 렌더링하지 않도록 최적화를 진행합니다. 하지만 Frustum Culling의 경우, 맵의 깊이가 깊어지거나 vertex가 복잡한 오브젝트가 시야 내에 있다면 그 성능이 급격하게 떨어지게 됩니다.

해당 프로젝트에서는 AI를 이용하여 구현한 3D mesh가 많았기에 vertex가 복잡한 오브젝트가 많았고, 맵의 크기도 상당하기에 Frustum Culling만으로는 충분한 최적화가 이루어지지 않았습니다.

반면 Occlusion culling의 경우, 특정 오브젝트가 다른 오브젝트에 의해 가려지면 렌더링에서 제외하는 방식이기 때문에 맵의 깊이가 깊어질 수록, 그리고 vertex가 복잡한 오브젝트가 다른 오브젝트에 많이 가려질 수록 효용성이 높아집니다.

해당 프로젝트에서는 오브젝트 간에 가려지는 경우가 많았고 vertex가 복잡한 오브젝트의 수도 많았으며 약 한달이라는 짧은 프로젝트 기간을 고려하였을 때, 개별 오브젝트에 LOD를 적용하는 것보다 공간 자체에 Occlusion Culling을 적용하는 것이 비용 대비 효과가 높고 일정을 맞추기에 적합할 것이라 판단하였습니다.

<div class="comparison-container" style="display: flex !important; flex-direction: row !important; justify-content: center !important; gap: 10px !important; margin: 30px 0 !important;">
  <div style="width: 48% !important; text-align: center !important;">
    <img src="{{ site.github.url }}/assets/source/mannequin/mannequin_Occlusion_Before.png" 
         style="width: 100% !important; height: auto !important; display: inline-block !important; border-radius: 5px;">
    <p style="font-size: 0.85em; color: #888; margin-top: 5px;">[그림 1] 적용 전 (15fps)</p>
  </div>
  <div style="width: 48% !important; text-align: center !important;">
    <img src="{{ site.github.url }}/assets/source/mannequin/mannequin_Occlusion_After.png" 
         style="width: 100% !important; height: auto !important; display: inline-block !important; border-radius: 5px;">
    <p style="font-size: 0.85em; color: #888; margin-top: 5px;">[그림 2] 적용 후 (60fps)</p>
  </div>
</div>

그리고 Occlusion Culling을 적용한 결과, 저희는 가장 프레임 드랍이 심한 곳을 기준으로 평균 15프레임에서 평균 60프레임으로 약 **300% 성능을 향상**시킬 수 있었습니다.

### Jitter 완화 작업

특정 증강을 획득했을 때 Jitter가 발생하는 것을 파악하여, 이를 중심으로 원인을 찾았습니다. 저희는 서버와 클라이언트가 서로 다른 시간축으로 상태를 각자 변경하고 있었기 때문에 여기에 네트워크 지연이 심할 경우 Jitter가 발생하는 것으로 판단하였습니다.

이에 상태값을 RPC를 통하여 한번 더 확정 짓는 방어코드를 추가하여 **동기화 기준을 서버로 일원화**하였고, 증강이 적용되는 시점을 의도적으로 동기화를 확인한 후로 미룸으로써 Jitter 발생을 안정화할 수 있었습니다.
