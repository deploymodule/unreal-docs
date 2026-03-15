# Cadence — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Cadence"` to your `Build.cs`.

## 2. Define a Combo

1. Right-click in Content Browser → **Miscellaneous → Data Asset** → `UCadenceComboData`
2. Configure:

| Field | Value |
|-------|-------|
| Combo ID | `Combo_UppercutFinisher` |
| Sequence | `["Attack", "Attack", "HeavyAttack"]` |
| Window Override | `1.2` (seconds) |
| Require Exact | `false` |
| Priority | `10` |

## 3. Add the Component

```cpp
// C++ constructor
CadenceComponent = CreateDefaultSubobject<UCadenceComponent>(TEXT("Cadence"));
```

## 4. Register Combos

```cpp
// Register at BeginPlay
CadenceComponent->RegisterCombo(UppercutComboData);
CadenceComponent->RegisterCombo(DashAttackComboData);

// Or register an entire array
CadenceComponent->RegisterCombos(AllCombos);
```

## 5. Push Input Events

Call `PushInput` from your input handlers (Enhanced Input, legacy, etc.):

```cpp
void AMyCharacter::OnAttackPressed()
{
    CadenceComponent->PushInput(FName("Attack"));
    // ... your other attack logic
}

void AMyCharacter::OnHeavyAttackPressed()
{
    CadenceComponent->PushInput(FName("HeavyAttack"));
}
```

## 6. Bind to OnComboMatched

```cpp
// C++
CadenceComponent->OnComboMatched.AddDynamic(this, &AMyCharacter::HandleComboMatched);

void AMyCharacter::HandleComboMatched(FName ComboID, UCadenceComboData* ComboData)
{
    if (ComboID == "Combo_UppercutFinisher")
    {
        // Execute the finisher ability
        ExecuteUppercutFinisher();
    }
}
```

In Blueprint, bind to the **On Combo Matched** event on the Cadence Component.

## 7. Cooldowns

Combos automatically enter a cooldown after firing to prevent repeat triggers:

```cpp
// Set per-combo cooldown on the data asset
ComboData->CooldownSeconds = 0.5f;

// Or query cooldown state
bool bReady = CadenceComponent->IsComboReady(FName("Combo_UppercutFinisher"));
```

## 8. Debug Visualization

```ini
; Show input history and match state in viewport
r.Cadence.Debug 1
```

This renders the recent input window and which combos are currently partially matched.
