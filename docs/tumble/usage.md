# Tumble — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Tumble"` to your `Build.cs`.

## 2. Add the Component

Add `UTumbleComponent` to your character class alongside a `UPhysicsAnimationComponent`:

```cpp
TumbleComponent = CreateDefaultSubobject<UTumbleComponent>(TEXT("Tumble"));
```

The Tumble component will automatically find and configure the `UPhysicsAnimationComponent` on the same actor.

## 3. Configure the Blend Profile

In the Blueprint details or C++, assign a `UTumbleBlendProfile` data asset:

| Region | Linear Drive Strength | Angular Drive Strength |
|--------|-----------------------|------------------------|
| Spine | 1000 | 1000 |
| Arms | 500 | 500 |
| Legs | 300 | 300 |
| Head | 200 | 200 |

## 4. Trigger Ragdoll

```cpp
// Enter ragdoll (e.g., on character death)
TumbleComponent->EnterRagdoll();

// With a specific cause impulse
TumbleComponent->EnterRagdoll(DeathImpulse, FName("spine_03"));
```

## 5. Apply Hit Reactions Without Ragdoll

```cpp
void AMyCharacter::OnHit(FVector ImpactDirection, float Damage, FName HitBone)
{
    float ImpulseStrength = FMath::Clamp(Damage * 10.f, 0.f, 2000.f);
    TumbleComponent->ApplyImpulse(HitBone, ImpactDirection * ImpulseStrength, EImpulseMode::Additive);
}
```

## 6. Configure Get-Up

```cpp
TumbleComponent->SetGetUpEnabled(true);
TumbleComponent->SetGetUpDelay(2.0f);               // seconds in ragdoll before auto get-up
TumbleComponent->SetFaceUpMontage(GetUpFaceUpAnim);
TumbleComponent->SetFaceDownMontage(GetUpFaceDownAnim);
```

## 7. Exit Ragdoll Manually

```cpp
// Immediately attempt get-up
TumbleComponent->ExitRagdoll();
```

## 8. Query Ragdoll State

```cpp
bool bInRagdoll = TumbleComponent->IsRagdolling();
ETumbleState State = TumbleComponent->GetState();
FVector PelvisLocation = TumbleComponent->GetRagdollPelvisLocation();
```

## 9. Bind to Events

```cpp
TumbleComponent->OnEnterRagdoll.AddDynamic(this, &AMyChar::OnEnterRagdoll);
TumbleComponent->OnExitRagdoll.AddDynamic(this, &AMyChar::OnExitRagdoll);
```
