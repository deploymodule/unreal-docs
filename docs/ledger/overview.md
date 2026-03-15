# Ledger — Overview

## Architecture

```
ULedgerSubsystem  (UGameInstanceSubsystem)
    └── TMap<FName, FLedgerFact>   FactDatabase

FLedgerFact  (struct)
    ├── ELedgerFactType  Type  (Bool / Int / Float / Name / String)
    └── union value field

ULedgerGateComponent  (UActorComponent)
    ├── TArray<FLedgerCondition>  Conditions
    ├── ELedgerGateLogic          Logic  (All / Any)
    └── OnGateOpened / OnGateClosed  Delegates

ULedgerBFL  (UBlueprintFunctionLibrary)
    └── Static wrappers for all ULedgerSubsystem functions
```

## Fact Types

| Type | C++ | Blueprint Pin | Example |
|------|-----|---------------|---------|
| `Bool` | `bool` | Boolean | `"BossDefeated" = true` |
| `Int` | `int32` | Integer | `"PlayerGold" = 1200` |
| `Float` | `float` | Float | `"PlayerHealth" = 87.5` |
| `Name` | `FName` | Name | `"ActiveFaction" = "Rebels"` |
| `String` | `FString` | String | `"LastSavePoint" = "Checkpoint_3A"` |

Facts are stored in a flat map keyed by `FName`. Writing a fact always overwrites the previous value. Facts not yet written return a typed default (false, 0, 0.0, NAME_None, "").

## Gate Component

`ULedgerGateComponent` monitors the fact database and evaluates its conditions whenever any watched fact changes. If all conditions (or any, depending on `Logic`) are met, the gate opens and `OnGateOpened` fires. If conditions are no longer met, `OnGateClosed` fires.

### Conditions

Each `FLedgerCondition` specifies:
- `FactKey` — which fact to watch
- `Comparison` — Equal, NotEqual, GreaterThan, LessThan, GreaterEqual, LessEqual
- `ExpectedValue` — the value to compare against

Example: "Gate opens when `BossDefeated == true` AND `PlayerGold >= 500`"

## Persistence

Facts survive level transitions (stored in the `UGameInstanceSubsystem`). For cross-session persistence, call `ULedgerSubsystem::SaveToSlot` / `LoadFromSlot`.

## Authority

Fact writes are always validated on the server. Clients can read facts locally (replicated on change); writes from clients trigger a server RPC.
