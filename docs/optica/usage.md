# Optica — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Optica"` to your `Build.cs`.

## 2. Add the Component

```cpp
OpticaComponent = CreateDefaultSubobject<UOpticaComponent>(TEXT("Optica"));
```

## 3. Assign a Weapon Sway Profile

Create a `UOpticaSwayProfile` data asset and assign:

| Property | Value |
|----------|-------|
| Lag Stiffness | 8.0 |
| Lag Damping | 0.7 |
| Tilt Max | 4.0 degrees |
| Vertical Max | 1.5 cm |

```cpp
OpticaComponent->SetSwayProfile(MySwayProfile);
```

## 4. Apply Recoil on Fire

Create a `UOpticaRecoilProfile` data asset:

| Property | Value |
|----------|-------|
| Camera Kick Up | 1.2 degrees |
| Camera Kick Right | 0.3 degrees (random) |
| Weapon Kick Back | 2.0 cm |
| Spring Stiffness | 12.0 |

```cpp
void AMyWeapon::Fire()
{
    OpticaComponent->ApplyRecoil(RecoilProfileAsset);
    // ... fire logic
}
```

## 5. Toggle ADS

```cpp
// Enter ADS
OpticaComponent->SetADS(true);
OpticaComponent->SetADSBlendSpeed(8.f);

// Exit ADS
OpticaComponent->SetADS(false);
```

## 6. Configure Breathing

```cpp
OpticaComponent->SetBreathingEnabled(true);
OpticaComponent->SetBreathingRate(0.25f);       // Hz (cycles per second)
OpticaComponent->SetBreathingAmplitude(0.8f);   // cm
```

Update exertion based on movement:

```cpp
// In Tick or movement callback
float Speed = GetCharacterMovement()->Velocity.Size();
OpticaComponent->SetExertionLevel(FMath::Clamp(Speed / 600.f, 0.f, 1.f));
```

## 7. Enable Sensor Emulation

```ini
r.Optica.Sensor.Enabled=1
r.Optica.Sensor.NoiseIntensity=0.03
r.Optica.Sensor.ExposureLag=0.4
r.Optica.Sensor.MotionBlurScale=2.0
```

## 8. Footstep Bob

```cpp
OpticaComponent->SetFootstepBobEnabled(true);
OpticaComponent->SetFootstepBobFrequency(2.0f);    // steps per second
OpticaComponent->SetFootstepBobAmplitude(0.5f);    // cm vertical
```
