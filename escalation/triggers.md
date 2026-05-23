# Escalation Mission — Full Trigger Chain Reference

Reading guide: `A → B, C` = "When A is completed/triggered, B and C start"
Object types:
- **Type 1 = DestroyUnits** (fires Outcomes when destruction condition is met)
- **Type 4 = WaitSeconds** (fires Outcomes after specified seconds)
- **Type 5 = SpawnUnit** (spawns unit)
- **Type 6 = CaptureAirbase** (capture condition)
- **Type 8 = CompleteOtherObjective / EndGame** (fires when AND condition is met)
- **Type 3 = ShowMessage**
- **Type 2 = StopOrCompleteObjective** (forces other objective to stop)

Faction mapping: **PALA = Primeva** (south, FleetCarrier1), **BDF = Boscali** (north, AssaultCarrier1)

---

## 0. Mission Start (`SetStartingConditions`)

23 objectives activate simultaneously right after mission load:

**Visible objectives (shown in UI) — 5**
- `Destroy_PALA_HWY1` (BDF executes)
- `Destroy_BDF_HWY2` (PALA executes)
- `Destroy_BDF_South` (PALA executes)
- `BDF_Victory_Conditions` (victory watcher, AND gate)
- `PALA_Victory_Conditions` (victory watcher, AND gate)

**Hidden watchers**
- `_BDF_Push`, `_PALA_WestPush`, `_PALA_EastPush` — backline-wipe tracking (no Outcomes, stats only)
- `DestroyDsrtDepots`, `DestroyMtnDepots`, `DestroyNorthDepots`, `DestroyCityDepots` — disable enemy reinforcements when depots are destroyed
- `ThreatenPALADsrtWest`, `ThreatenPALADsrtEast`, `ThreatenPALAMtnWest`, `ThreatenPALAMtnEast` — PALA territory threat detection
- `ThreatenBDFNorthWest`, `ThreatenBDFNorthEast`, `ThreatenBDFCityWest`, `ThreatenBDFCityEast` — BDF territory threat detection
- `DestroyAllCriticalBDFFacilities`, `DestroyAllCriticalPALAFacilities` — watcher for full aircraft-factory wipeout
- `SinkBDFCarrierWithNotification`, `SinkPALACarrierWithNotification` — carrier sunk notification
- `DamageHighValueTargets` — starts carrier-arrival countdown when any single HVT in the list is destroyed (CompleteAny mode)

---

## 1. Visible Objective Cascade (BDF destroys PALA line)

```
Destroy_PALA_HWY1                   destroyed → Start_Destroy_PALA_LVFac1
Destroy_PALA_LVFac1                 destroyed → Start_Destroy_PALA_AirbaseDesert
Destroy_PALA_AirbaseDesert          destroyed → Start_Destroy_PALA_LVFac2
Destroy_PALA_LVFac2                 destroyed → Start_Destroy_PALA_AirFacRear
Destroy_PALA_AirFacRear             destroyed → Start_Destroy_PALA_SupportFac
                                              → Start_Destroy_PALA_Enrichment
Destroy_PALA_SupportFac             destroyed → Start_Destroy_PALA_TankFac
Destroy_PALA_TankFac                destroyed → Start_Destroy_PALA_AirbaseMtn
Destroy_PALA_AirbaseMtn             destroyed → Start_Destroy_PALA_AirFacMtn
Destroy_PALA_AirFacMtn              destroyed → Start_Destroy_PALA_AirFacIsland
Destroy_PALA_AirFacIsland           destroyed → Start_Destroy_PALA_AirbaseIsland
Destroy_PALA_AirbaseIsland          destroyed → (none — end of line)
```

Additional branches:
```
Start_Destroy_PALA_SupportFac       → starts Destroy_PALA_SupportFac
                                    + starts BDF_CapFarm (capture option: The Farm)
Start_Destroy_PALA_Enrichment       → starts DestroyPALAEnrichment
                                    + starts BDF_CapSouthEP (capture option: EnrichmentPlantSouth)
```

---

## 2. Visible Objective Cascade (PALA destroys BDF line)

```
Destroy_BDF_South                   destroyed → Start_Destroy_BDF_LVFac1
Destroy_BDF_HWY2                    destroyed → Start_Destroy_BDF_LVFac2
Destroy_BDF_LVFac1                  destroyed → Start_Destroy_BDF_AirFac1
Destroy_BDF_LVFac2                  destroyed → Start_Destroy_BDF_AirFac2
Destroy_BDF_AirFac1                 destroyed → Start_Destroy_BDF_AirbaseNorth
Destroy_BDF_AirFac2                 destroyed → Start_Destroy_BDF_AirbaseCity
Destroy_BDF_AirbaseNorth            destroyed → Start_Destroy_BDF_SupportFac
                                              → Start_Destroy_BDF_Enrichment
Destroy_BDF_AirbaseCity             destroyed → Start_Destroy_BDF_TankFac
                                              → Start_Destroy_BDF_Enrichment
Destroy_BDF_TankFac                 destroyed → Start_Destroy_BDF_SupportFac
                                              → Start_Destroy_BDF_AirbaseCity
Destroy_BDF_SupportFac              destroyed → Start_Destroy_BDF_TankFac
                                              → Start_Destroy_BDF_AirFac1
```

Additional branches:
```
Start_Destroy_BDF_LVFac2            → starts Destroy_BDF_LVFac2
                                    + starts PALA_CapMaris (capture option: Maris Heliport)
Start_Destroy_BDF_Enrichment        → starts DestroyBDFEnrichment
                                    + starts PALA_CapNorthEP (capture option: EnrichmentPlantNorth)
```

**Important**: BDF line has bidirectional dependencies (`SupportFac ↔ TankFac ↔ AirbaseCity`). Destroying any one of them activates the others.

---

## 3. Territory Threat Detection → Reinforcement Call (15-min delay)

Each Threaten\* triggers when its designated building list crosses the completion threshold. The `Data[0]` field is the `CompleteOrder` enum (0=Any, 1=All, 2=InOrder, 3=Some) and `Data[1]` is `completeSomePercent` (only meaningful in `CompleteSome` mode). "Destruction" of a building = the building's `disabled` flag becomes true (i.e. its HP reaches 0); the 0.5 percent value is **list-percent**, not per-building HP.

```
ThreatenBDFCityEast      triggered → SendReinf_CityEast      → Reinf_FromCityToEast (Wait 900s)      → SpawnConvoyFromCityToEastFront      (15 units)
ThreatenBDFCityWest      triggered → SendReinf_CityWest      → Reinf_FromCityToWest (Wait 900s)      → SpawnConvoyFromCityToWestFront      (15 units)
ThreatenBDFNorthEast     triggered → SendReinf_NorthEast     → Reinf_FromNorthToEast (Wait 900s)     → SpawnConvoyFromNorthToEastFront     (15 units)
ThreatenBDFNorthWest     triggered → SendReinf_NorthWest     → Reinf_FromNorthToWest (Wait 900s)     → SpawnConvoyFromNorthToWestFront     (15 units)

ThreatenPALADsrtEast     triggered → SendReinf_DesertEast    → Reinf_FromDesertToEast (Wait 900s)    → SpawnConvoyFromDesertToEastFront    (15 units)
ThreatenPALADsrtWest     triggered → SendReinf_DesertWest    → Reinf_FromDesertToWest (Wait 900s)    → SpawnConvoyFromDesertToWestFront    (15 units)
ThreatenPALAMtnEast      triggered → SendReinf_MountainEast  → Reinf_FromMountainToEast (Wait 900s)  → SpawnConvoyFromMountainToEastFront  (15 units)
ThreatenPALAMtnWest      triggered → SendReinf_MountainWest  → Reinf_FromMountainToWest (Wait 900s)  → SpawnConvoyFromMountainToWestFront  (15 units)
```

**Sequence**: threat detected → immediately SendReinf → 15-min wait → 15-unit convoy spawn (e.g. `PALAConvDsrtEast1~15`)

---

## 4. Rear Depot Destruction → Enemy Reinforcement Cutoff

Each Destroy\*Depots triggers when its RearDepot pair is destroyed. (`Data[0]=1.0`=CompleteAll, `Data[1]=0.5`=ignored in All mode. Both `RearDepot1` and `RearDepot2` must be `disabled` — i.e. HP=0.)

```
DestroyCityDepots    destroyed → StopCityReinforcements    → cancels Reinf_FromCityToEast/West
DestroyNorthDepots   destroyed → StopNorthReinforcements   → cancels Reinf_FromNorthToEast/West
DestroyMtnDepots     destroyed → StopMtnReinforcements     → cancels Reinf_FromMountainToEast/West
DestroyDsrtDepots    destroyed → StopDesertReinforcements  → cancels Reinf_FromDesertToEast/West
```

| Depot pair | Destroyer | Cancelled convoys |
|---|---|---|
| City_RearDepot1+2 | PALA | City→East/West reinforcements |
| North_RearDepot1+2 | PALA | North→East/West reinforcements |
| Mountain_RearDepot1+2 | BDF | Mountain→East/West reinforcements |
| Desert_RearDepot1+2 | BDF | Desert→East/West reinforcements |

**Bottom line**: destroying a RearDepot pair permanently cancels that region's convoys. Even if Threaten has already fired and the 15-min countdown started, the convoy won't arrive if you wipe the depot pair before it spawns.

---

## 5. Aircraft Carrier Spawn Trigger

```
DamageHighValueTargets (any 1 of HVT list is destroyed — CompleteAny mode)
   ↓
StartNavalReinfMessageDelay
   ↓
NavalReinfMessageDelay (Wait 50s)
   ↓
   ├─→ NavalIncomingMessage: "Naval reinforcements are enroute. ETA 10 minutes."
   └─→ StartSendNavalReinf
         ↓
       SendNavalReinf (Wait 600s = 10 min)
         ↓
         ├─→ SpawnFleets: BDF_Carrier + PALA_Carrier + BDF_Esc1/2 + PALA_Esc1/2 spawn simultaneously
         └─→ SpawnFleetsMessage
```

**Total delay**: from HVT damage → **50 + 600 = 650s (≈10 min 50 sec)** until both carriers spawn together

---

## 6. Carrier Sunk Notification (UI message)

```
SinkBDFCarrierWithNotification  → on BDF_Carrier sunk  → BDFCarrierSunkMessage  ("The BDF Fleet Carrier has been sunk.")
SinkPALACarrierWithNotification → on PALA_Carrier sunk → PALACarrierSunkMessage ("The PALA Fleet Carrier has been sunk.")
```

(`SinkBDFCarrier` / `SinkPALACarrier` themselves have no message — they only act as AND inputs into Victory_Conditions.)

---

## 7. Victory Cascade

```
DestroyAllCriticalBDFFacilities (CityFac1~9 + NorthFac1~9 wiped)
   ↓
StartSinkBDFCarrier
   ↓
SinkBDFCarrier objective activated

DestroyAllCriticalPALAFacilities (IslandFac1~3 + MtnFac1~8 + RearFac1~6 wiped)
   ↓
StartSinkPALACarrier
   ↓
SinkPALACarrier objective activated
```

```
PALA_Victory_Conditions (AND gate)
   ├─ DestroyAllCriticalBDFFacilities complete?
   └─ SinkBDFCarrier complete?
   if both YES →
       ├─→ Victory (EndGame)
       └─→ Victory_Message ("The enemy has capitulated. We are victorious!")

BDF_Victory_Conditions (AND gate)
   ├─ DestroyAllCriticalPALAFacilities complete?
   └─ SinkPALACarrier complete?
   if both YES →
       ├─→ Victory (EndGame)
       └─→ Victory_Message
```

**Key**: the Sink\*Carrier objective only activates **after** DestroyAllCritical\*Facilities is complete. In other words, you must wipe every enemy aircraft factory before the SinkCarrier objective becomes active. (You can still physically sink the carrier earlier, but the victory gate requires both objectives to be in "complete" state.)

---

## 8. Capture Option Triggers (auxiliary)

```
Start_Destroy_BDF_LVFac2        →  also starts PALA_CapMaris (Maris Heliport)
Start_Destroy_PALA_SupportFac   →  also starts BDF_CapFarm (The Farm)
Start_Destroy_BDF_Enrichment    →  also starts PALA_CapNorthEP (EnrichmentPlantNorth)
Start_Destroy_PALA_Enrichment   →  also starts BDF_CapSouthEP (EnrichmentPlantSouth)
```

Capture flips the airbase to your side immediately. No Outcomes attached (doesn't influence cascade, just gives a forward airfield).

---

## 9. Quick Reference: "When X is destroyed, what starts?"

### PALA buildings BDF needs to destroy

| Destroy | Next objective(s) triggered |
|---|---|
| PALA HWY1 (hwy1_SPAAG1_1 + 6 vehicles) | Destroy_PALA_LVFac1 |
| PALA LVFac1 (palalvf1_factory_\* × 4) | Destroy_PALA_AirbaseDesert |
| PALA AirbaseDesert (Dsrt_VehicleDepot, Dsrt_pillbox, etc. × 7) | Destroy_PALA_LVFac2 |
| PALA LVFac2 (palalvf2_factory_\* × 4) | Destroy_PALA_AirFacRear |
| PALA AirFacRear (RearFac1~6) | **Destroy_PALA_SupportFac + DestroyPALAEnrichment + BDF_CapSouthEP + BDF_CapFarm** |
| PALA SupportFac (factory_large_24/29/30, factory_tall/\_1) | Destroy_PALA_TankFac |
| PALA TankFac (factory_large_11/12/13) | Destroy_PALA_AirbaseMtn |
| PALA AirbaseMtn (Mtn_HLT-M_11, factory_tall_3/4, MtnFac1/2) | Destroy_PALA_AirFacMtn |
| PALA AirFacMtn (MtnFac4~8) | Destroy_PALA_AirFacIsland |
| PALA AirFacIsland (IslandFac1~3) | Destroy_PALA_AirbaseIsland |
| PALA AirbaseIsland (R9Isle_HLT, SPAAG, Linebreaker, etc.) | (end of line) |
| **All IslandFac1~3 + MtnFac1~8 + RearFac1~6** | **SinkPALACarrier activates** → BDF wins on sink |

### BDF buildings PALA needs to destroy

| Destroy | Next objective(s) triggered |
|---|---|
| BDF South (SB_SPAAG1, pillbox_6, VehicleDepot1, HLT-FT_6, etc. × 8) | Destroy_BDF_LVFac1 |
| BDF HWY2 (hwy2_SPAAG1_2 + 5 units) | **Destroy_BDF_LVFac2 + PALA_CapMaris** |
| BDF LVFac1 (bdflvf1_factory_\* × 4) | Destroy_BDF_AirFac1 |
| BDF LVFac2 (bdflvf2_factory_\* × 4) | Destroy_BDF_AirFac2 |
| BDF AirFac1 (NorthFac5~10) | Destroy_BDF_AirbaseNorth |
| BDF AirFac2 (CityFac1/2/4/5/9) | Destroy_BDF_AirbaseCity |
| BDF AirbaseNorth (North_RearDepot1/2, NorthFac1~4) | **Destroy_BDF_SupportFac + DestroyBDFEnrichment + PALA_CapNorthEP** |
| BDF AirbaseCity (City_RearDepot2, CityFac3/6/7/8) | **Destroy_BDF_TankFac + DestroyBDFEnrichment + PALA_CapNorthEP** |
| BDF TankFac (BDF_MBTFac1~4) | Destroy_BDF_SupportFac + Destroy_BDF_AirbaseCity |
| BDF SupportFac (BDF_SuppFac1~3, factory_tall_2/5) | Destroy_BDF_TankFac + Destroy_BDF_AirFac1 |
| **All CityFac1~9 + NorthFac1~9** | **SinkBDFCarrier activates** → PALA wins on sink |

### Rear Depots (shared pattern, both sides)

| Destroy | Result |
|---|---|
| Either RearDepot1+2 pair (City / North / Mtn / Desert) | That region's convoys permanently cancelled |

### HVT Damage

| Destroy (any 1 HVT) | Result |
|---|---|
| Any single CityFac / NorthFac / MtnFac / RearFac / IslandFac / factory / RearDepot / Enrichment | Message after 50s → both carriers spawn 10 min later |
