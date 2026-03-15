# Cadence — API Reference

## UCadenceComponent

`#include "CadenceComponent.h"`

### Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `PushInput` | `void PushInput(FName InputID)` | Submit an input event to history |
| `RegisterCombo` | `void RegisterCombo(UCadenceComboData* Combo)` | Register a combo for matching |
| `UnregisterCombo` | `void UnregisterCombo(UCadenceComboData* Combo)` | Remove a combo |
| `RegisterCombos` | `void RegisterCombos(TArray<UCadenceComboData*> Combos)` | Register multiple combos |
| `UnregisterAll` | `void UnregisterAll()` | Clear all registered combos |
| `IsComboReady` | `bool IsComboReady(FName ComboID)` | Check if combo cooldown has elapsed |
| `GetPartialProgress` | `float GetPartialProgress(FName ComboID)` | 0–1 progress through combo sequence |
| `ClearHistory` | `void ClearHistory()` | Flush the input history window |
| `SetWindowSeconds` | `void SetWindowSeconds(float Seconds)` | Override global input window |

### Delegates

| Delegate | Signature | Fired when |
|----------|-----------|------------|
| `OnComboMatched` | `(FName ComboID, UCadenceComboData* Data)` | A registered combo sequence is detected |

## UCadenceComboData

`#include "CadenceComboData.h"`

| Property | Type | Description |
|----------|------|-------------|
| `ComboID` | FName | Unique combo identifier |
| `Sequence` | TArray\<FName\> | Ordered input IDs to match |
| `WindowOverride` | float | Per-combo time window (0 = global) |
| `bRequireExact` | bool | No extra inputs allowed between steps |
| `Priority` | int32 | Higher value = checked first on conflict |
| `CooldownSeconds` | float | Post-fire cooldown duration |

## CVars

| CVar | Type | Default | Description |
|------|------|---------|-------------|
| `r.Cadence.WindowSeconds` | float | 1.5 | Global input history window |
| `r.Cadence.PrefixDelay` | float | 0.15 | Delay before firing prefix combos |
| `r.Cadence.Debug` | int | 0 | Show debug input history overlay |
