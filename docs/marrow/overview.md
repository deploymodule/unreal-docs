# Marrow — Overview

## System Modules

Marrow is organized into independent subsystems:

### 1. Sanity System (`UMarrowSanityComponent`)

Tracks player sanity as a 0–100 float. Sanity drains when the player is exposed to horror stimuli (dark zones, monster proximity, disturbing events) and recovers when safe. Configurable drain rates, thresholds, and recovery curves. Fires delegates at configurable thresholds (e.g., `OnSanityLow`, `OnSanityBroken`).

### 2. Dread System (`UMarrowDreadComponent`)

Separate from sanity, dread is a short-term tension meter that spikes on sudden scares and rapidly decays. Used to drive heartbeat audio, camera shake, and controller rumble. Feeds the `FMarrowDreadCurve` which drives blend layers in the animation graph.

### 3. Perception AI (`UMarrowPerceptionComponent`)

Extended perception for horror NPCs:
- **Hearing** — range-based with material occlusion
- **Vision** — cone-based with light-level sensitivity (dim light = shorter range)
- **Vibration** — footstep and proximity sensing
- **Scent** — tag-based scent markers that persist in world space

### 4. Environmental Horror (`UMarrowEnvironmentComponent`)

Monitors the player's environment for horror conditions:
- Darkness level (samples scene luma at camera)
- Isolation (time since last contact with a friendly actor)
- Temperature (tied to environment zones)
- Sound level (ambient + reactive)

Outputs a composite `HorrorPressure` float (0–1) consumed by Sanity and Dread.

### 5. Flicker Manager (`UMarrowFlickerSubsystem`)

Coordinates light flicker behavior for horror ambience. Lights can be registered and assigned flicker profiles (frequency, variance, on-chance, fail patterns). Avoids random independent timers by batching all flicker updates in one subsystem tick.

## Inter-System Communication

```
EnvironmentComponent → HorrorPressure → SanityComponent → SanityDrain
                                       → DreadComponent → DreadSpike
PerceptionComponent  → OnPlayerDetected → (your game logic)
```

Each system exposes delegates — coupling is always opt-in.
