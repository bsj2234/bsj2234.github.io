---
layout: portfolio_item
title: "보스 전투 시스템 최적화 프로젝트"
description: "보스 공격 모듈화 및 성능 최적화를 통한 게임플레이 향상"
game_type: "액션"
project_type: "개인 프로젝트"
focus: "보스 전투 시스템 모듈화, 최적화, 협업경험"
image: "https://lh3.googleusercontent.com/d/13--bB7a81Kv06HgrtOf4U5ao6TZDEG7G"
date: 2024-10-17
order: 5
---

<!-- 목차 -->
<div id="toc" class="toc-container">
    <h2>목차</h2>
    <ul>
        <li><a href="#project-overview">프로젝트 개요</a></li>
        <li><a href="#key-achievements">주요 성과</a>
            <ul>
                <li><a href="#collaboration">협업 경험</a></li>
                <ul>
                    <li><a href="#jira">지라 일정관리</a></li>
                    <li><a href="#git">깃 머지 충돌 해결</a></li>
                </ul>
                <li><a href="#boss-attack-modularization">보스 공격 모듈화</a></li>
                <li><a href="#performance-optimization">성능 최적화</a></li>
            </ul>
        </li>
        <li><a href="#tech-stack">기술 스택</a></li>
        <li><a href="#collaboration">협업 경험</a></li>
        <li><a href="#conclusion">결론</a></li>
    </ul>
</div>

<!-- 유튜브 영상 삽입 -->
<div class="video-container">
    <iframe 
        src="https://www.youtube.com/embed/tjLz6Xdfl2Y" 
        title="보스 전투 시스템 데모" 
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" 
        allowfullscreen>
    </iframe>
</div>

<h2 id="project-overview">프로젝트 개요</h2>

이 프로젝트에서는 액션 게임의 기획팀과의 협업이 중심이었습니다. 기업의 요구사항을 만족하는 기획 사항을 기획팀에서 작성해 제공하고 그를 토대로 게임을 개발하는 것이 목표였습니다.

<h2 id="key-achievements">주요 성과</h2>
이 과정에서 깃 파일관리, 지라를 통한 일정관리, 또 기획팀의 편의를 위한 개발로 협업 경험을 쌓을 수 있었습니다.

<h3 id="collaboration">1. 협업 경험</h3>
<h4 id="jira">지라 일정관리</h4>
![지라 일정관리 예시](https://lh3.googleusercontent.com/d/1Xx5Ewg4kX-t-dOj3vr0N1TT6lnu1U9kA)
지라를 통한 일정 관리로 프로젝트의 진행상황과 진행도를 한눈에 파악하기에 좋았습니다. 또한 앞으로 개발해야할 항목들을 한눈에 볼 수 있어 낭비되는 시간이 적어졌습니다.

<h4 id="git">깃 머지 충돌 해결</h4>
![깃 머지 충돌 해결 예시](https://lh3.googleusercontent.com/d/1rp8yKubtFkL-0kSUS-4E1FXYqP_o6cl1)
깃에서 머지 충돌이 발생한 경우 손머지를 통해 충돌을 해결하는 경험을 할 수 있었습니다.
이를 통해 프리팹을 포함한 여러 에셋들도 손머지를 통해 해결할 수 있다는 점과 메타데이터가 손상되거나 중복되는경우 이전의 메타데이터를 복구함으로서 문제를 해결할 수 있다는 점을 알 수 있었습니다.

<h3 id="boss-attack-modularization">2. 보스 공격 모듈화</h3>

보스의 공격 패턴을 개별 모듈로 분리하여 다음과 같은 이점을 얻었습니다:

- 새로운 공격 패턴을 쉽게 추가하고 기존 패턴을 수정할 수 있는 유연성 확보
- 코드의 가독성과 유지보수성 향상
- 다양한 보스 캐릭터 간 공격 패턴 재사용 가능

<!-- 이미지 플레이스홀더 -->
![보스 공격 모듈 다이어그램](https://lh3.googleusercontent.com/d/1yNb3N3VOf7sbEepW1fNkJw8V2A6NEYLC)

<h3 id="performance-optimization">3. 성능 최적화</h3>

1. 공격 패턴 선정 과정에서 처리 속도개선 테스트를 진행 했습니다:

- `stackalloc`과 `Span<T>`을 활용하여 힙 할당을 줄임
- 임시 배열 대신 스택 메모리 사용으로 가비지 컬렉션 부하 감소
- 결과적으로 이 부분의 성능이 40% 향상됨

2. 대기 중인 적에서 비헤이비어 디자이너가 리소스를 사용하던 부분을 개선했습니다
- 프로파일러를 통한 문제 파악
- 적이 다수 존재하는 씬에서 40FPS -> 80FPS의 성능 향상

<div class="code-block-container">
{% highlight csharp %}
using EnumTypes;
using System.Collections.Generic;
using UnityEngine;

[CreateAssetMenu(fileName = "Boss_Both_UltimateAttackModule", menuName = "Enemy/AttackModule/Boss_Both_UltimateAttack")]
public class SO_Boss_Both_UltimateAttackModule : SO_AttackModule
{
    public GameObject Prefab_areaAttack;
    public GameObject Prefab_Spike;
    public float MaxArea = 50f;
    public float SpikeGap = 4f;
    public float SpikeRandomOffset = 2f;
    public float SpikeProbability = .3f;

    public override void StartAttack(EnemyBase owner, int type)
    {

        switch (type)
        {
            case 0:
                // 이펙트
                owner.StartVFX("Boss_Teleport_End1");
                owner.StartVFX("Boss_Teleport_End2");

                // 공격 영역 생성
                Transform targetTrf = owner.transform;
                Vector3 targetPos = targetTrf.position;
                targetPos.y = 0f;
                GameObject attackArea_GO = GameObject.Instantiate(Prefab_areaAttack,
                    targetPos, Quaternion.identity);
                BossDoubleAreaAttack areaAttack = attackArea_GO.GetComponent<BossDoubleAreaAttack>();
                areaAttack.Init(Damage, owner.Attack.RangeTypeThreshold, IsClose(owner));
                owner.Attack.CurrentProjectile = areaAttack;

                // 이전 스파이크 삭제
                SpikeManager.Instance.DestroyAllSpike();
                SpikeManager.Instance.DestroyAllTrash();

                // 새로운 스파이크 생성
                SpawnMultipleSpikeInAreaAndStore(owner, Prefab_Spike, MaxArea,
                SpikeGap, SpikeProbability, SpikeRandomOffset,
                owner.Attack.CurrentSpikeSpawners);

                break;
            case 1:
                // 공격 영역 트리거
                owner.Attack.CurrentProjectile.Trigger();

                // 스파이크 트리거
                foreach (SpikeSpawner spike in owner.Attack.CurrentSpikeSpawners)
                {
                    spike.Trigger();
                }
                owner.Attack.CurrentSpikeSpawners.Clear();

                // 새로운 스파이크 생성
                SpawnMultipleSpikeInAreaAndStore(owner, Prefab_Spike, MaxArea,
                SpikeGap, SpikeProbability, SpikeRandomOffset,
                owner.Attack.CurrentSpikeSpawners);
                break;
            case 2:
                // 공격 영역 트리거
                owner.Attack.CurrentProjectile.Trigger();

                // 스파이크 트리거
                foreach (SpikeSpawner spike in owner.Attack.CurrentSpikeSpawners)
                {
                    spike.Trigger();
                }

                // 스파이크 초기화
                owner.Attack.CurrentProjectile = null;
                owner.Attack.CurrentSpikeSpawners.Clear();
                break;
        }

    }

    public override void StartAttackMove(EnemyBase owner, int type)
    {
        // 이펙트
        owner.StartVFX("Boss_Teleport");

        switch (type)
        {
            case 0:
                // 위로 캐릭터 발사
                owner.Move.Launch(Vector3.up * 100);
                break;
            case 1:
                // 중앙으로 이동
                owner.transform.position = owner.SpawnedPosition + Vector3.up * 15;

                // 아래로 캐릭터 발사
                owner.Move.Launch(-Vector3.up * 100);
                break;
        }
    }

    private void SpawnMultipleSpikeInAreaAndStore(EnemyBase owner, GameObject spike, float areaRadius, float gap,
        float probability, float spikeRandomOffset, List<SpikeSpawner> spikeContainer)
    {
        // 사각 순회하며 원 내부만 체크 후 스폰
        Vector3 centerOffset = new Vector3(areaRadius * .5f, 0f, areaRadius * .5f);
        for (int i = 0; i < areaRadius / gap; i++)
            for (int k = 0; k < areaRadius / gap; k++)
            {
                if (Random.value < probability)
                {
                    Vector3 offset = new Vector3(i * gap, 0f, k * gap) - centerOffset;
                    Vector3 pos = owner.transform.position + offset;
                    if (Vector3.Distance(pos, owner.transform.position) < areaRadius * .5f)
                    {
                        SpikeSpawner curSpike = SpawnSpike(owner, spike, pos, spikeRandomOffset);
                        spikeContainer.Add(curSpike);
                        SpikeManager.Instance.Spikes.Add(curSpike);
                    }
                }
            }
    }

    private AttackRangeType IsClose(EnemyBase owner)
    {
        return owner.Attack.GetAttackRangeType();
    }

    private SpikeSpawner SpawnSpike(EnemyBase owner, GameObject spike, Vector3 position, float spikeRandomOffset)
    {
        Vector3 targetPos = position;
        targetPos.y = 0f;
        targetPos += new Vector3(Random.value * spikeRandomOffset, 0f, Random.value * spikeRandomOffset);
        GameObject projectileObject = ObjectPoolManager.Instance.DequeueObject(spike);
        projectileObject.transform.position = targetPos;
        projectileObject.transform.rotation = Quaternion.identity;


        SpikeSpawner spikeInst = projectileObject.GetComponent<SpikeSpawner>();
        spikeInst.Init(false);

        return spikeInst;
    }
}
{% endhighlight %}
</div>

<h2 id="tech-stack">기술 스택</h2>

- C#
- Unity Engine

<h2 id="conclusion">결론</h2>

이 프로젝트를 통해 기업, 기획팀과의 협업을 통해 실무에 조금 더 가까워진 것 같습니다. 게임의 중요 시스템 중 하나인 보스 전투 메커니즘을 모듈화 해 유연하게 사용 가능했습니다. 최적화를 통해 더 부드러운 게임 환경을 제작 할 수 있었습니다.
