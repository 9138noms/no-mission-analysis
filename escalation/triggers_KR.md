# Escalation 트리거 체인 전수 정리

읽는 법: `A → B, C` = "A 완료/발동 시 B와 C 시작"
오브젝트 Type:
- **Type 1 = DestroyUnits** (파괴 조건 충족 시 Outcomes 발동)
- **Type 4 = WaitSeconds** (지정 초 대기 후 Outcomes 발동)
- **Type 5 = SpawnUnit** (유닛 스폰)
- **Type 6 = CaptureAirbase** (점령 조건)
- **Type 8 = CompleteOtherObjective / EndGame** (AND 조건 충족 시 발동)
- **Type 3 = ShowMessage**
- **Type 2 = StopOrCompleteObjective** (다른 오브젝트 강제 중단)

---

## 0. 미션 시작 (`SetStartingConditions`)

미션 로드 직후 23개 오브젝트가 동시에 활성화:

**가시 오브젝티브 (UI 표시) — 5개**
- `Destroy_PALA_HWY1` (BDF가 수행)
- `Destroy_BDF_HWY2` (PALA가 수행)
- `Destroy_BDF_South` (PALA가 수행)
- `BDF_Victory_Conditions` (승리 감시기, AND 게이트)
- `PALA_Victory_Conditions` (승리 감시기, AND 게이트)

**숨겨진 감시기 (Hidden)**
- `_BDF_Push`, `_PALA_WestPush`, `_PALA_EastPush` — 백라인 와이프 추적용 (Outcome 없음, 통계 수집만)
- `DestroyDsrtDepots`, `DestroyMtnDepots`, `DestroyNorthDepots`, `DestroyCityDepots` — 데포 파괴 시 적 증원 중단
- `ThreatenPALADsrtWest`, `ThreatenPALADsrtEast`, `ThreatenPALAMtnWest`, `ThreatenPALAMtnEast` — PALA 영토 위협 감시
- `ThreatenBDFNorthWest`, `ThreatenBDFNorthEast`, `ThreatenBDFCityWest`, `ThreatenBDFCityEast` — BDF 영토 위협 감시
- `DestroyAllCriticalBDFFacilities`, `DestroyAllCriticalPALAFacilities` — 모든 항공기 공장 박멸 감시
- `SinkBDFCarrierWithNotification`, `SinkPALACarrierWithNotification` — 항모 격침 알림용
- `DamageHighValueTargets` — HVT 리스트 중 1개라도 destroyed(disabled=HP 0) 시 항모 등장 카운트다운 (CompleteAny 모드)

---

## 1. 가시 오브젝티브 캐스케이드 (BDF가 PALA 박멸하는 라인)

```
Destroy_PALA_HWY1                   파괴 →  Start_Destroy_PALA_LVFac1
Destroy_PALA_LVFac1                 파괴 →  Start_Destroy_PALA_AirbaseDesert
Destroy_PALA_AirbaseDesert          파괴 →  Start_Destroy_PALA_LVFac2
Destroy_PALA_LVFac2                 파괴 →  Start_Destroy_PALA_AirFacRear
Destroy_PALA_AirFacRear             파괴 →  Start_Destroy_PALA_SupportFac
                                         →  Start_Destroy_PALA_Enrichment
Destroy_PALA_SupportFac             파괴 →  Start_Destroy_PALA_TankFac
Destroy_PALA_TankFac                파괴 →  Start_Destroy_PALA_AirbaseMtn
Destroy_PALA_AirbaseMtn             파괴 →  Start_Destroy_PALA_AirFacMtn
Destroy_PALA_AirFacMtn              파괴 →  Start_Destroy_PALA_AirFacIsland
Destroy_PALA_AirFacIsland           파괴 →  Start_Destroy_PALA_AirbaseIsland
Destroy_PALA_AirbaseIsland          파괴 →  (없음 — 라인 끝)
```

추가 분기:
```
Start_Destroy_PALA_SupportFac       →  Destroy_PALA_SupportFac 시작
                                    +  BDF_CapFarm 시작 (The Farm 캡쳐 옵션)
Start_Destroy_PALA_Enrichment       →  DestroyPALAEnrichment 시작
                                    +  BDF_CapSouthEP 시작 (EnrichmentPlantSouth 캡쳐 옵션)
```

---

## 2. 가시 오브젝티브 캐스케이드 (PALA가 BDF 박멸하는 라인)

```
Destroy_BDF_South                   파괴 →  Start_Destroy_BDF_LVFac1
Destroy_BDF_HWY2                    파괴 →  Start_Destroy_BDF_LVFac2
Destroy_BDF_LVFac1                  파괴 →  Start_Destroy_BDF_AirFac1
Destroy_BDF_LVFac2                  파괴 →  Start_Destroy_BDF_AirFac2
Destroy_BDF_AirFac1                 파괴 →  Start_Destroy_BDF_AirbaseNorth
Destroy_BDF_AirFac2                 파괴 →  Start_Destroy_BDF_AirbaseCity
Destroy_BDF_AirbaseNorth            파괴 →  Start_Destroy_BDF_SupportFac
                                         →  Start_Destroy_BDF_Enrichment
Destroy_BDF_AirbaseCity             파괴 →  Start_Destroy_BDF_TankFac
                                         →  Start_Destroy_BDF_Enrichment
Destroy_BDF_TankFac                 파괴 →  Start_Destroy_BDF_SupportFac
                                         →  Start_Destroy_BDF_AirbaseCity
Destroy_BDF_SupportFac              파괴 →  Start_Destroy_BDF_TankFac
                                         →  Start_Destroy_BDF_AirFac1
```

추가 분기:
```
Start_Destroy_BDF_LVFac2            →  Destroy_BDF_LVFac2 시작
                                    +  PALA_CapMaris 시작 (Maris Heliport 캡쳐 옵션)
Start_Destroy_BDF_Enrichment        →  DestroyBDFEnrichment 시작
                                    +  PALA_CapNorthEP 시작 (EnrichmentPlantNorth 캡쳐 옵션)
```

**중요**: BDF 라인은 양방향 의존성 있음 (`SupportFac ↔ TankFac ↔ AirbaseCity`). 한 쪽만 부숴도 다른 쪽 오브젝티브가 활성화됨.

---

## 3. 영토 위협 감지 → 증원 호출 (15분 지연)

각 Threaten*는 지정된 빌딩이 destroyed (disabled=HP 0) 시 발동. Data[0]은 `CompleteOrder` enum (0=Any, 1=All, 2=InOrder, 3=Some), Data[1]은 `completeSomePercent` (CompleteSome 모드에서만 의미). 0.5는 빌딩별 HP가 아닌 **리스트 퍼센트**임.

```
ThreatenBDFCityEast      파괴 →  SendReinf_CityEast      →  Reinf_FromCityToEast (Wait 900s)      →  SpawnConvoyFromCityToEastFront      (15 유닛)
ThreatenBDFCityWest      파괴 →  SendReinf_CityWest      →  Reinf_FromCityToWest (Wait 900s)      →  SpawnConvoyFromCityToWestFront      (15 유닛)
ThreatenBDFNorthEast     파괴 →  SendReinf_NorthEast     →  Reinf_FromNorthToEast (Wait 900s)     →  SpawnConvoyFromNorthToEastFront     (15 유닛)
ThreatenBDFNorthWest     파괴 →  SendReinf_NorthWest     →  Reinf_FromNorthToWest (Wait 900s)     →  SpawnConvoyFromNorthToWestFront     (15 유닛)

ThreatenPALADsrtEast     파괴 →  SendReinf_DesertEast    →  Reinf_FromDesertToEast (Wait 900s)    →  SpawnConvoyFromDesertToEastFront    (15 유닛)
ThreatenPALADsrtWest     파괴 →  SendReinf_DesertWest    →  Reinf_FromDesertToWest (Wait 900s)    →  SpawnConvoyFromDesertToWestFront    (15 유닛)
ThreatenPALAMtnEast      파괴 →  SendReinf_MountainEast  →  Reinf_FromMountainToEast (Wait 900s)  →  SpawnConvoyFromMountainToEastFront  (15 유닛)
ThreatenPALAMtnWest      파괴 →  SendReinf_MountainWest  →  Reinf_FromMountainToWest (Wait 900s)  →  SpawnConvoyFromMountainToWestFront  (15 유닛)
```

**시퀀스**: 위협 감지 → 즉시 SendReinf → 15분 대기 → 호송대 15기 스폰 (예: PALAConvDsrtEast1~15)

---

## 4. 후방 데포 파괴 → 적 증원 차단

각 Destroy*Depots는 해당 지역 RearDepot 페어가 destroyed (disabled=HP 0) 시 발동. (Data[0]=1.0=CompleteAll → 두 개 다 disabled 필요, Data[1]=0.5는 CompleteAll에서 무시됨)

```
DestroyCityDepots    파괴 →  StopCityReinforcements    →  Reinf_FromCityToEast/West 중단
DestroyNorthDepots   파괴 →  StopNorthReinforcements   →  Reinf_FromNorthToEast/West 중단
DestroyMtnDepots     파괴 →  StopMtnReinforcements     →  Reinf_FromMountainToEast/West 중단
DestroyDsrtDepots    파괴 →  StopDesertReinforcements  →  Reinf_FromDesertToEast/West 중단
```

| 데포 페어 | 파괴자 | 차단되는 호송대 |
|---|---|---|
| City_RearDepot1+2 | PALA | City→East/West 보강 |
| North_RearDepot1+2 | PALA | North→East/West 보강 |
| Mountain_RearDepot1+2 | BDF | Mountain→East/West 보강 |
| Desert_RearDepot1+2 | BDF | Desert→East/West 보강 |

**즉, RearDepot 2개 박살내면 그 지역의 호송대 영구 차단**. Threaten이 발동되어 15분 카운트 시작했어도 그 사이에 RearDepot 박살내면 호송대 안 옴.

---

## 5. 항공모함 등장 트리거

```
DamageHighValueTargets (HVT 리스트 1개라도 destroyed — CompleteAny 모드)
   ↓
StartNavalReinfMessageDelay
   ↓
NavalReinfMessageDelay (Wait 50s)
   ↓
   ├─→ NavalIncomingMessage: "Naval reinforcements are enroute. ETA 10 minutes."
   └─→ StartSendNavalReinf
         ↓
       SendNavalReinf (Wait 600s = 10분)
         ↓
         ├─→ SpawnFleets: BDF_Carrier + PALA_Carrier + BDF_Esc1/2 + PALA_Esc1/2 동시 스폰
         └─→ SpawnFleetsMessage
```

**총 지연**: HVT 데미지 → **50 + 600 = 650초 (≈10분 50초)** 후 양측 항모 동시 등장

---

## 6. 항모 격침 알림 (UI 메시지용)

```
SinkBDFCarrierWithNotification  → BDF_Carrier 격침 시  → BDFCarrierSunkMessage  ("The BDF Fleet Carrier has been sunk.")
SinkPALACarrierWithNotification → PALA_Carrier 격침 시 → PALACarrierSunkMessage ("The PALA Fleet Carrier has been sunk.")
```

(별도로 SinkBDFCarrier / SinkPALACarrier는 메시지 없이 Victory_Conditions의 AND 입력으로만 사용됨)

---

## 7. 승리 캐스케이드

```
DestroyAllCriticalBDFFacilities (CityFac1~9 + NorthFac1~9 박멸)
   ↓
StartSinkBDFCarrier
   ↓
SinkBDFCarrier 오브젝티브 활성화

DestroyAllCriticalPALAFacilities (IslandFac1~3 + MtnFac1~8 + RearFac1~6 박멸)
   ↓
StartSinkPALACarrier
   ↓
SinkPALACarrier 오브젝티브 활성화
```

```
PALA_Victory_Conditions (AND 게이트)
   ├─ DestroyAllCriticalBDFFacilities 완료?
   └─ SinkBDFCarrier 완료?
   둘 다 YES →
       ├─→ Victory (EndGame)
       └─→ Victory_Message ("The enemy has capitulated. We are victorious!")

BDF_Victory_Conditions (AND 게이트)
   ├─ DestroyAllCriticalPALAFacilities 완료?
   └─ SinkPALACarrier 완료?
   둘 다 YES →
       ├─→ Victory (EndGame)
       └─→ Victory_Message
```

**중요**: 항모 격침 가능 시점은 `DestroyAllCritical*Facilities` 완료 **이후**. 즉 적 모든 항공기 공장을 먼저 박살내야 SinkCarrier 오브젝티브가 활성화됨. (실제로는 카운트만 안 될 뿐 항모는 이전에도 격침 가능하지만, 승리 조건 게이트는 두 오브젝티브가 둘 다 "complete" 상태여야 함)

---

## 8. 캡쳐 옵션 트리거 (보조)

```
Start_Destroy_BDF_LVFac2        →  PALA_CapMaris 동시 시작 (Maris Heliport)
Start_Destroy_PALA_SupportFac   →  BDF_CapFarm 동시 시작 (The Farm)
Start_Destroy_BDF_Enrichment    →  PALA_CapNorthEP 동시 시작 (EnrichmentPlantNorth)
Start_Destroy_PALA_Enrichment   →  BDF_CapSouthEP 동시 시작 (EnrichmentPlantSouth)
```

캡쳐는 점령 즉시 자기편 비행장으로 전환. Outcomes 없음 (체인에 영향 X, 전선 비행장만 확보).

---

## 9. 빠른 참조표: "X 파괴 시 무엇이 시작되는가"

### BDF가 부숴야 할 PALA 빌딩

| 부수면 | 시작되는 다음 오브젝티브 |
|---|---|
| PALA HWY1 (hwy1_SPAAG1_1 + 6개 차량) | Destroy_PALA_LVFac1 |
| PALA LVFac1 (palalvf1_factory_* 4개) | Destroy_PALA_AirbaseDesert |
| PALA AirbaseDesert (Dsrt_VehicleDepot, Dsrt_pillbox 등 7개) | Destroy_PALA_LVFac2 |
| PALA LVFac2 (palalvf2_factory_* 4개) | Destroy_PALA_AirFacRear |
| PALA AirFacRear (RearFac1~6) | **Destroy_PALA_SupportFac + DestroyPALAEnrichment + BDF_CapSouthEP + BDF_CapFarm** |
| PALA SupportFac (factory_large_24/29/30, factory_tall/_1) | Destroy_PALA_TankFac |
| PALA TankFac (factory_large_11/12/13) | Destroy_PALA_AirbaseMtn |
| PALA AirbaseMtn (Mtn_HLT-M_11, factory_tall_3/4, MtnFac1/2) | Destroy_PALA_AirFacMtn |
| PALA AirFacMtn (MtnFac4~8) | Destroy_PALA_AirFacIsland |
| PALA AirFacIsland (IslandFac1~3) | Destroy_PALA_AirbaseIsland |
| PALA AirbaseIsland (R9Isle_HLT, SPAAG, Linebreaker 등) | (라인 끝) |
| **모든 IslandFac1~3 + MtnFac1~8 + RearFac1~6** | **SinkPALACarrier 활성화** → 격침 시 BDF 승리 |

### PALA가 부숴야 할 BDF 빌딩

| 부수면 | 시작되는 다음 오브젝티브 |
|---|---|
| BDF South (SB_SPAAG1, pillbox_6, VehicleDepot1, HLT-FT_6 등 8개) | Destroy_BDF_LVFac1 |
| BDF HWY2 (hwy2_SPAAG1_2 + 5개) | **Destroy_BDF_LVFac2 + PALA_CapMaris** |
| BDF LVFac1 (bdflvf1_factory_* 4개) | Destroy_BDF_AirFac1 |
| BDF LVFac2 (bdflvf2_factory_* 4개) | Destroy_BDF_AirFac2 |
| BDF AirFac1 (NorthFac5~10) | Destroy_BDF_AirbaseNorth |
| BDF AirFac2 (CityFac1/2/4/5/9) | Destroy_BDF_AirbaseCity |
| BDF AirbaseNorth (North_RearDepot1/2, NorthFac1~4) | **Destroy_BDF_SupportFac + DestroyBDFEnrichment + PALA_CapNorthEP** |
| BDF AirbaseCity (City_RearDepot2, CityFac3/6/7/8) | **Destroy_BDF_TankFac + DestroyBDFEnrichment + PALA_CapNorthEP** |
| BDF TankFac (BDF_MBTFac1~4) | Destroy_BDF_SupportFac + Destroy_BDF_AirbaseCity |
| BDF SupportFac (BDF_SuppFac1~3, factory_tall_2/5) | Destroy_BDF_TankFac + Destroy_BDF_AirFac1 |
| **모든 CityFac1~9 + NorthFac1~9** | **SinkBDFCarrier 활성화** → 격침 시 PALA 승리 |

### 후방 데포 (양측 공통 패턴)

| 부수면 | 결과 |
|---|---|
| 양측 RearDepot1+2 (City/North/Mtn/Desert 중 하나의 페어) | 해당 지역 호송대 영구 차단 |

### HVT 손상

| 부수면 (HVT 1개라도 destroyed) | 결과 |
|---|---|
| CityFac/NorthFac/MtnFac/RearFac/IslandFac/공장/RearDepot/Enrichment 중 어느 1개 | 50초 후 메시지 → 10분 후 양측 항모 동시 스폰 |
