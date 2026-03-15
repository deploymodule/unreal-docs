# Cadence — Overview

## Architecture

```
UCadenceComponent  (UActorComponent)
    ├── TArray<UCadenceComboData*>   RegisteredCombos
    ├── TArray<FCadenceInputEvent>   InputHistory        — rolling window
    ├── FCadenceMatcher              Matcher             — pattern evaluation
    └── OnComboMatched               Delegate            — fired on match
```

## How Pattern Matching Works

### Input Events

Every input that Cadence should track is submitted via `PushInput(FName InputID)`. The `InputID` is a project-defined name (e.g., `"Attack"`, `"Dodge"`, `"Jump"`). Cadence does not read from the Enhanced Input system directly — you call `PushInput` from your own input handlers. This keeps the system decoupled and works with any input backend.

### Input History Window

Cadence maintains a rolling history of input events, each stamped with a timestamp. The history window duration is configurable (`r.Cadence.WindowSeconds`, default 1.5s). Events older than the window are discarded. The window size determines how fast a combo must be executed.

### Combo Data Assets

Each combo is defined as a `UCadenceComboData` asset:

| Property | Description |
|----------|-------------|
| `ComboID` | Unique name for this combo |
| `Sequence` | Ordered array of `FName` input IDs |
| `WindowOverride` | Per-combo time window (0 = use global default) |
| `bRequireExact` | If true, no extra inputs allowed between sequence steps |
| `Priority` | Higher priority combos are checked first |

### Pattern Evaluation

After each `PushInput`, the matcher evaluates all registered combos against the current history:

1. Filter the history to the combo's time window
2. Extract the ordered input IDs from the filtered history
3. Check if the combo sequence appears as a subsequence (or exact match if `bRequireExact`)
4. If matched, fire `OnComboMatched` with the `ComboID` and mark the combo on cooldown

### Conflict Resolution

If multiple combos match simultaneously, the highest-priority combo wins. Combos that are prefixes of longer combos have their fire delayed by `r.Cadence.PrefixDelay` seconds to give the longer combo a chance to complete.

## Design Philosophy

Cadence does one thing: detect sequences. It does not:
- Execute abilities
- Drive animations
- Consume inputs
- Replicate (combos are detected client-side; you replicate the result)

This keeps the system minimal, testable, and easy to integrate with any execution system.
