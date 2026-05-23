# Nuclear Option — Mission Analysis

Static analysis of [Nuclear Option](https://store.steampowered.com/app/2168680/Nuclear_Option/) built-in mission files. Trigger chains, objective dependencies, win conditions — reverse-engineered from decompiled mission JSON.

No strategy commentary, no exploit notes. Just the raw structure for anyone studying mission flow.

## Missions

| Mission | Interactive graph | Reference docs |
|---|---|---|
| **Escalation** | [Open diagram](https://9138noms.github.io/no-mission-analysis/escalation/triggers.html) | [triggers.md](escalation/triggers.md) · [objectives.md](escalation/objectives.md) · [Korean version](escalation/triggers_KR.md) |

## How the analysis was produced

1. Built-in missions are not loose `.json` files on disk — they are packed into Unity `resources.assets` as `TextAsset`s, loaded at runtime via `Resources.LoadAll<TextAsset>("Missions")` (see [`MissionGroup.cs`](https://github.com/) in the decompiled `Assembly-CSharp`).
2. A small BepInEx plugin ([NOMissionDumper](https://github.com/9138noms/NOMissionDumper)) dumps every `TextAsset` as-is to disk — byte-exact, no `JsonUtility.ToJson` round-trip and no auto version upgrade. This matters because **saving via the mission editor re-serializes through `JsonUtility` and applies `MissionVersionUpgrade.Upgrade()` on load**, so editor-copied files differ from the original in field order, float precision, and sometimes structure. Download the prebuilt dll from the [releases page](https://github.com/9138noms/NOMissionDumper/releases).
3. Trigger/outcome graphs are extracted directly from the dumped JSON's `objectives.Objectives` and `objectives.Outcomes` lists.

## Key notes about the trigger format

Each `Objective` of type `DestroyUnits` has two leading data slots before the unit list:

```
Data[0].FloatValue → CompleteOrder enum (0=Any, 1=All, 2=InOrder, 3=Some)
Data[1].FloatValue → completeSomePercent  (only meaningful when CompleteOrder == Some)
Data[2..]          → target unit UniqueNames
```

The `0.5` value that appears in nearly every objective is **list-percent, not per-unit HP**. A unit counts as "destroyed" when its `disabled` flag becomes true (i.e. HP reaches 0); the percent then says "what fraction of the list must be disabled for the objective to complete" — and only when `CompleteOrder` is `Some`.

In practice, across all 38 built-in missions, `CompleteOrder == Some` is never used — the percent slider is effectively dead. The most common mode is `InOrder` (units must be destroyed in list order).

## Repository layout

```
no-mission-analysis/
├── README.md
└── escalation/
    ├── triggers.html         # interactive diagram (self-contained, offline)
    ├── triggers.md           # full trigger chain reference (English)
    ├── triggers_KR.md        # Korean version
    └── objectives.md         # objective list + win conditions + carrier/enrichment system
```

## License

Analysis text and diagrams: CC0 / public domain — use freely, credit not required.
Mission data is © Shockfront Studios.
