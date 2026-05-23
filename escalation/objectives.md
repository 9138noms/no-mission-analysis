# Escalation 미션 오브젝티브 정리 (NOCS 준비)

분석 대상: `Missions\Escalation decomfile\Escalation decomfile.json` (1.2MB, 39563 lines)
맵: Heartland (Escalation 표준 8v8/대규모 미션)
파벌 매핑: **PALA = Primeva** (남쪽, FleetCarrier1), **BDF = Boscali** (북쪽, AssaultCarrier1)

---

## 1. 미션 메타 (settings)

| 항목 | 값 |
|---|---|
| 설명 | 모든 비행장 활성, 항모 중반 등장, 적 항공기 공장 전멸 + 적 항모 격침이 승리 조건 |
| `allowRespawn` | true |
| `rankMultiplier` | 0.75 |
| `successfulSortieBonus` | 0.75 |
| **`nuclearEscalationThreshold`** | **625** (팀 점수 도달 시 전술핵 해금) |
| **`strategicEscalationThreshold`** | **1225** (전략핵 해금) |
| `minRankTacticalWarhead` / `Strategic` | 0 / 0 (해금되면 누구나 쏠 수 있음) |
| `startingWarheads` / `reserve` | 16 / 16 (양측 동일) |

---

## 2. 승리 조건 (확정)

```
PALA_Victory_Conditions  ← AND
  ├─ DestroyAllCriticalBDFFacilities
  │     (CityFac1~9, NorthFac1~9 — 즉 BDF 항공기 공장 전부)
  └─ SinkBDFCarrier (BDF_Carrier 격침)

BDF_Victory_Conditions  ← AND
  ├─ DestroyAllCriticalPALAFacilities
  │     (IslandFac1~3, MtnFac1~8, RearFac1~6 — PALA 항공기 공장 전부)
  └─ SinkPALACarrier (PALA_Carrier 격침)
```

둘 다 충족하면 `Victory` (EndGame Type=8) + "The enemy has capitulated. We are victorious!" 메시지

---

## 3. 가시 오브젝티브 트리 (BDF 측 = Boscali가 수행)

```
START
├─ Destroy_PALA_HWY1       [Capture Airstrip]   ← 미션 시작 즉시
│     └→ Destroy_PALA_LVFac1   [Destroy Vehicle Factories]
│           └→ Destroy_PALA_AirbaseDesert   [Neutralise Airbase]
│                 └→ Destroy_PALA_LVFac2   [Destroy Vehicle Factories]
│                       └→ Destroy_PALA_AirFacRear   [Destroy Aircraft Factories]
│                             ├→ Destroy_PALA_SupportFac   [Destroy Support Vehicle Factories]
│                             │     └→ Destroy_PALA_TankFac   [Destroy Armour Factories]
│                             │           └→ Destroy_PALA_AirbaseMtn   [Neutralise Airbase]
│                             │                 └→ Destroy_PALA_AirFacMtn   [Destroy Aircraft Factories]
│                             │                       └→ Destroy_PALA_AirFacIsland
│                             │                             └→ Destroy_PALA_AirbaseIsland
│                             └→ DestroyPALAEnrichment   [Destroy Enrichment Plant]
│                                + BDF_CapSouthEP (점령 옵션, EnrichmentPlantSouth)
```

---

## 4. 가시 오브젝티브 트리 (PALA 측 = Primeva가 수행)

```
START
├─ Destroy_BDF_HWY2        [Capture Airstrip]   ← 미션 시작 즉시
│     └→ Destroy_BDF_LVFac2   [Destroy Vehicle Factories]
│           + PALA_CapMaris (점령 옵션, Maris Heliport)
│           └→ Destroy_BDF_AirFac2   [Destroy Aircraft Factories]
│                 └→ Destroy_BDF_AirbaseCity   [Neutralise Airbase]
│                       └→ Destroy_BDF_TankFac   [Destroy Armour Factories]
│                             ├→ Destroy_BDF_SupportFac   [Destroy Support Vehicle Factories]
│                             │     └→ Destroy_BDF_AirFac1   [Destroy Aircraft Factories]
│                             │           └→ Destroy_BDF_AirbaseNorth   [Neutralise Airbase]
│                             │                 ├→ Destroy_BDF_SupportFac (재진입)
│                             │                 └→ DestroyBDFEnrichment + PALA_CapNorthEP
│                             └→ DestroyBDFEnrichment   [Destroy Enrichment Plant]
│                                + PALA_CapNorthEP (점령 옵션, EnrichmentPlantNorth)
│
└─ Destroy_BDF_South       [Capture Airbase]   ← 미션 시작 즉시
      └→ Destroy_BDF_LVFac1   [Destroy Vehicle Factories]
            └→ Destroy_BDF_AirFac1   [Destroy Aircraft Factories]
                  └→ Destroy_BDF_AirbaseNorth
                        └→ ...
```

구조 메모: PALA는 South + HWY2 **2개 라인 동시** 시작. BDF는 HWY1 **1개 라인** 시작.

---

## 5. 항공모함 (중반 스폰)

트리거 체인:
```
DamageHighValueTargets (HVT 1개라도 50% 피해)
   ↓
NavalReinfMessageDelay (50초)
   ↓
"Naval reinforcements are enroute. ETA 10 minutes." 메시지
   ↓
SendNavalReinf (600초 대기 = 10분)
   ↓
SpawnFleets: BDF_Carrier + PALA_Carrier + BDF_Esc1/2 + PALA_Esc1/2 동시 스폰
```

- **BDF_Carrier**: `AssaultCarrier1` (작은 강습항모), Boscali, 위치 `(-9181, 5.8, 70221)` — 북동쪽 바다, CaptureStrength 50
- **PALA_Carrier**: `FleetCarrier1` (큰 함대항모), Primeva, 위치 `(-58797, 6.2, -41850)` — 남서쪽 바다, CaptureStrength 100
- 양측 호위함 `Corvette1` × 2 (BDF_Esc1/2, PALA_Esc1/2)

### HVT 리스트 (DamageHighValueTargets에 등록된 빌딩, 50% HP 피해로 카운트)
BDF: CityFac1~9, NorthFac1~9, bdflvf1/2_factory_* (large/tall), BDF_MBTFac1~4, BDF_SuppFac1~3, City_RearDepot1/2, North_RearDepot1/2, BDF_enrichmentPlant1
PALA: MtnFac1~8, RearFac1~6 (RearFac1/2/3/4/5), IslandFac1~3, palalvf1/2_factory_*, factory_large_11/12/13/20/24/29/30, factory_tall/_1/_2/_3/_4/_5, Desert_RearDepot1/2, Mountain_RearDepot1/2, PALA_enrichmentPlant1

---

## 6. 농축 시설 (핵 생산)

| UniqueName | Faction | DisplayName | productionType | 시간 |
|---|---|---|---|---|
| BDF_enrichmentPlant1 | Boscali | (Enrichment Plant) | **Nuclear Warhead** | 145초/발 |
| PALA_enrichmentPlant1 | Primeva | (Enrichment Plant) | **Nuclear Warhead** | 145초/발 |
| EnrichmentPlantNorth | Boscali | North Coast Enrichment Plant | (capturable airbase) | CaptureDefense=10 |
| EnrichmentPlantSouth | Primeva | South Coast Enrichment Plant | (capturable airbase) | CaptureDefense=10 |

좌표:
- EnrichmentPlantNorth (BDF, 캡쳐 시 PALA 소유): `(2852, 3.0, 35160)`
- EnrichmentPlantSouth (PALA, 캡쳐 시 BDF 소유): `(7004, 2.4, -35508)`

---

## 7. Capture 가능한 보조 비행장 (Type=6 CaptureAirbase)

| UniqueName | Faction | Target | 트리거되는 시점 |
|---|---|---|---|
| PALA_CapMaris | Primeva | Maris Heliport | Start_Destroy_BDF_LVFac2 와 같이 |
| BDF_CapFarm | Boscali | The Farm | Start_Destroy_PALA_SupportFac 와 같이 |
| PALA_CapNorthEP | Primeva | EnrichmentPlantNorth | Start_Destroy_BDF_Enrichment 와 같이 |
| BDF_CapSouthEP | Boscali | EnrichmentPlantSouth | Start_Destroy_PALA_Enrichment 와 같이 |

---

## 8. 숨겨진 (Hidden) 이벤트 오브젝티브

UI에 표시되지 않음. 트리거 전용.

### 8.1 위협 감지 → 적 증원 호출 (Threaten*)

| UniqueName | Faction | 감시 대상 | 발동 시 |
|---|---|---|---|
| ThreatenBDFCityEast | Primeva | hwy2/lvf2 동쪽 라인 빌딩 50% | SendReinf_CityEast |
| ThreatenBDFCityWest | Primeva | lvf2 서쪽 + Corvette1_7 | SendReinf_CityWest |
| ThreatenBDFNorthEast | Primeva | hwy2/City/CityFac 동쪽 | SendReinf_NorthEast |
| ThreatenBDFNorthWest | Primeva | lvf1 서쪽 라인 | SendReinf_NorthWest |
| ThreatenPALADsrtEast | Boscali | (Desert 동쪽 빌딩들) | SendReinf_DsrtEast |
| ThreatenPALADsrtWest | Boscali | (Desert 서쪽 빌딩들) | SendReinf_DsrtWest |
| ThreatenPALAMtnEast | Boscali | (Mountain 동쪽 빌딩들) | SendReinf_MtnEast |
| ThreatenPALAMtnWest | Boscali | (Mountain 서쪽 빌딩들) | SendReinf_MtnWest |

### 8.2 증원 호송대 (Reinf_*)

각각 `WaitSeconds 900` (15분) 후 호송대 스폰:
- Reinf_FromDesertToEast / DesertToWest (Primeva)
- Reinf_FromMountainToEast / MountainToWest (Primeva)
- Reinf_FromCityToEast / CityToWest (Boscali)
- Reinf_FromNorthToEast / NorthToWest (Boscali)

### 8.3 깊은 푸시 트래커 (_*_Push)

| UniqueName | Faction | 내용 |
|---|---|---|
| _PALA_EastPush | Primeva | BDF 동쪽 모든 depot/factory 리스트 (~22개) |
| _PALA_WestPush | Primeva | BDF 서쪽 모든 depot 리스트 |
| _BDF_Push | Boscali | PALA 전 영토 depot/factory 리스트 (~22개) |

이건 깊숙히 침투해서 백라인을 와이프했는지 추적하는 메트릭 오브젝티브. Outcome은 비어 있음 (집계용으로 추정).

### 8.4 항모 알림 변형

- SinkBDFCarrierWithNotification → "The BDF Fleet Carrier has been sunk." 메시지 출력
- SinkPALACarrierWithNotification → "The PALA Fleet Carrier has been sunk." 메시지 출력

(SinkBDFCarrier / SinkPALACarrier 본체는 메시지 없이 PALA/BDF_Victory_Conditions의 AND 입력)

### 8.5 depot 처리 미션 (Destroy* depot 단독)

DestroyDsrtDepots, DestroyMtnDepots, DestroyNorthDepots, DestroyCityDepots — 각 지역 depot 박멸 시 StopXxxReinforcements (적 증원 차단) 발동.

---

## 9. 가시 오브젝티브 전체 리스트 (UI 표시)

### BDF 측 (Boscali, "Destroy PALA *")
1. Destroy_PALA_HWY1 — Capture Airstrip
2. Destroy_PALA_LVFac1 — Destroy Vehicle Factories
3. Destroy_PALA_AirbaseDesert — Neutralise Airbase
4. Destroy_PALA_LVFac2 — Destroy Vehicle Factories
5. Destroy_PALA_AirFacRear — Destroy Aircraft Factories
6. Destroy_PALA_SupportFac — Destroy Support Vehicle Factories
7. Destroy_PALA_TankFac — Destroy Armour Factories
8. Destroy_PALA_AirbaseMtn — Neutralise Airbase
9. Destroy_PALA_AirFacMtn — Destroy Aircraft Factories
10. Destroy_PALA_AirFacIsland — Destroy Aircraft Factories
11. Destroy_PALA_AirbaseIsland — Neutralise Airbase
12. DestroyPALAEnrichment — Destroy Enrichment Plant
13. SinkPALACarrier — Sink Fleet Carrier (중반 스폰 후)

### PALA 측 (Primeva, "Destroy BDF *")
1. Destroy_BDF_HWY2 — Capture Airstrip
2. Destroy_BDF_South — Capture Airbase
3. Destroy_BDF_LVFac1 — Destroy Vehicle Factories
4. Destroy_BDF_LVFac2 — Destroy Vehicle Factories
5. Destroy_BDF_AirFac1 — Destroy Aircraft Factories
6. Destroy_BDF_AirFac2 — Destroy Aircraft Factories
7. Destroy_BDF_AirbaseCity — Neutralise Airbase
8. Destroy_BDF_AirbaseNorth — Neutralise Airbase
9. Destroy_BDF_TankFac — Destroy Armour Factories
10. Destroy_BDF_SupportFac — Destroy Support Vehicle Factories
11. DestroyBDFEnrichment — Destroy Enrichment Plant
12. SinkBDFCarrier — Sink Fleet Carrier (중반 스폰 후)
