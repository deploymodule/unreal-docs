# Marrow — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Marrow"` to your `Build.cs`.

## 2. Sanity System

Add `UMarrowSanityComponent` to the player character:

```cpp
SanityComponent = CreateDefaultSubobject<UMarrowSanityComponent>(TEXT("Sanity"));
SanityComponent->MaxSanity = 100.f;
SanityComponent->RecoveryRate = 2.f;    // per second in safe area
```

Drain sanity on horror events:

```cpp
SanityComponent->DrainSanity(15.f);          // sudden scare
SanityComponent->SetDrainActive(true, 5.f);  // continuous drain at 5/sec
```

Bind to threshold events:

```cpp
SanityComponent->OnSanityThreshold.AddDynamic(this, &AMyChar::OnSanityChanged);
```

## 3. Dread System

```cpp
DreadComponent = CreateDefaultSubobject<UMarrowDreadComponent>(TEXT("Dread"));

// Spike dread on a sudden scare
DreadComponent->SpikeD read(80.f);

// Get current dread (0–100) for audio/VFX driven systems
float CurrentDread = DreadComponent->GetDread();
```

## 4. Perception AI

Add to NPC actors:

```cpp
MarrowPerception = CreateDefaultSubobject<UMarrowPerceptionComponent>(TEXT("MarrowPerception"));
MarrowPerception->HearingRange = 800.f;
MarrowPerception->VisionConeAngle = 60.f;
MarrowPerception->LightSensitivity = 1.5f;  // extra sensitive in bright light
```

Bind the player detected event:

```cpp
MarrowPerception->OnPlayerDetected.AddDynamic(this, &AMyNPC::OnPlayerDetected);
MarrowPerception->OnPlayerLost.AddDynamic(this, &AMyNPC::OnPlayerLost);
```

## 5. Environmental Horror

```cpp
EnvironmentComponent = CreateDefaultSubobject<UMarrowEnvironmentComponent>(TEXT("MarrowEnv"));

// Get composite horror pressure (0–1) to drive other systems
float Pressure = EnvironmentComponent->GetHorrorPressure();
SanityComponent->SetDrainActive(Pressure > 0.3f, Pressure * 8.f);
```

## 6. Flicker Manager

Register lights for flicker:

```cpp
UMarrowFlickerSubsystem* Flicker = GetWorld()->GetSubsystem<UMarrowFlickerSubsystem>();
Flicker->RegisterLight(HallwayLight, FlickerProfileData);
Flicker->SetFlickerEnabled(HallwayLight, true);
```
