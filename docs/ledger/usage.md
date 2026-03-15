# Ledger — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Ledger"` to your `Build.cs`.

## 2. Write Facts

From C++:

```cpp
ULedgerSubsystem* Ledger = GetGameInstance()->GetSubsystem<ULedgerSubsystem>();

Ledger->SetBool(FName("BossDefeated"), true);
Ledger->SetInt(FName("PlayerGold"), 1200);
Ledger->SetFloat(FName("CompletionPercent"), 73.5f);
Ledger->SetName(FName("ActiveFaction"), FName("Rebels"));
```

From Blueprint, use the **Ledger** category functions in the BFL:
- `Ledger Set Bool`
- `Ledger Set Int`
- `Ledger Set Float`

## 3. Read Facts

```cpp
bool bDefeated = Ledger->GetBool(FName("BossDefeated"));
int32 Gold     = Ledger->GetInt(FName("PlayerGold"));
float Percent  = Ledger->GetFloat(FName("CompletionPercent"));
```

## 4. Check Fact Existence

```cpp
if (Ledger->HasFact(FName("BossDefeated")))
{
    // The fact has been written at least once
}
```

## 5. Delete a Fact

```cpp
Ledger->RemoveFact(FName("TemporaryFlag"));
```

## 6. Gate Component Setup

Add `ULedgerGateComponent` to any actor (door, trigger, NPC, etc.):

1. In Blueprint, add the **Ledger Gate** component
2. In the details panel, add conditions:

| Fact Key | Comparison | Value |
|----------|------------|-------|
| `BossDefeated` | Equal | `true` |
| `PlayerGold` | GreaterEqual | `500` |

3. Set **Logic** to `All` (both must be true)
4. Bind to **On Gate Opened** and **On Gate Closed** events

```cpp
LedgerGate->OnGateOpened.AddDynamic(this, &AMyDoor::OpenDoor);
LedgerGate->OnGateClosed.AddDynamic(this, &AMyDoor::CloseDoor);
```

## 7. One-Shot Facts

A common pattern is "fire this event only once":

```cpp
if (!Ledger->GetBool(FName("IntroShown")))
{
    Ledger->SetBool(FName("IntroShown"), true);
    ShowIntroSequence();
}
```

## 8. Persistence

```cpp
Ledger->SaveToSlot(FName("MainSave"));
Ledger->LoadFromSlot(FName("MainSave"));
```
