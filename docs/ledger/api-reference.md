# Ledger — API Reference

## ULedgerSubsystem

`#include "LedgerSubsystem.h"`

### Write Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `SetBool` | `void SetBool(FName Key, bool Value)` | Write a boolean fact |
| `SetInt` | `void SetInt(FName Key, int32 Value)` | Write an integer fact |
| `SetFloat` | `void SetFloat(FName Key, float Value)` | Write a float fact |
| `SetName` | `void SetName(FName Key, FName Value)` | Write a name fact |
| `SetString` | `void SetString(FName Key, FString Value)` | Write a string fact |

### Read Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `GetBool` | `bool GetBool(FName Key, bool Default)` | Read a boolean fact |
| `GetInt` | `int32 GetInt(FName Key, int32 Default)` | Read an integer fact |
| `GetFloat` | `float GetFloat(FName Key, float Default)` | Read a float fact |
| `GetName` | `FName GetName(FName Key, FName Default)` | Read a name fact |
| `GetString` | `FString GetString(FName Key, FString Default)` | Read a string fact |

### Management Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `HasFact` | `bool HasFact(FName Key)` | Check if fact has been written |
| `RemoveFact` | `void RemoveFact(FName Key)` | Delete a fact |
| `ClearAll` | `void ClearAll()` | Clear entire database |
| `GetAllKeys` | `TArray<FName> GetAllKeys()` | All fact keys |
| `SaveToSlot` | `void SaveToSlot(FName SlotName)` | Persist to save game |
| `LoadFromSlot` | `void LoadFromSlot(FName SlotName)` | Restore from save game |

### Delegates

| Delegate | Signature | Fired when |
|----------|-----------|------------|
| `OnFactChanged` | `(FName Key, FLedgerFact OldValue, FLedgerFact NewValue)` | Any fact is written |

## ULedgerGateComponent

`#include "LedgerGateComponent.h"`

| Property | Type | Description |
|----------|------|-------------|
| `Conditions` | TArray\<FLedgerCondition\> | List of fact conditions |
| `Logic` | ELedgerGateLogic | All (AND) or Any (OR) |
| `OnGateOpened` | FSimpleMulticastDelegate | Fired when gate conditions become true |
| `OnGateClosed` | FSimpleMulticastDelegate | Fired when gate conditions become false |

### Methods

| Function | Signature | Description |
|----------|-----------|-------------|
| `IsOpen` | `bool IsOpen()` | Current gate state |
| `ForceEvaluate` | `void ForceEvaluate()` | Re-evaluate conditions immediately |

## FLedgerCondition

| Field | Type | Description |
|-------|------|-------------|
| `FactKey` | FName | Fact to compare |
| `Comparison` | ELedgerComparison | Equal/NotEqual/GreaterThan/LessThan/etc. |
| `BoolValue` | bool | Expected bool value |
| `IntValue` | int32 | Expected int value |
| `FloatValue` | float | Expected float value |
| `NameValue` | FName | Expected name value |
