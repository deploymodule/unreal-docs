# Relay — Usage

## Step 1: Enable the Plugin

**Edit > Plugins > Relay** — enable and restart.

## Step 2: Broadcast a Message

=== "Blueprint"
    - Call **Relay > Broadcast** with a GameplayTag and an optional payload struct.

=== "C++"
    ```cpp
    #include "Relay/RelayBlueprintLibrary.h"

    // Simple broadcast (no payload)
    URelayBlueprintLibrary::Broadcast(this, FGameplayTag::RequestGameplayTag("Event.Combat.MeleeHit"));

    // Broadcast with a payload
    FRelayPayload_Combat Payload;
    Payload.DamageAmount = 42.f;
    Payload.HitLocation = ImpactPoint;
    URelayBlueprintLibrary::BroadcastWithPayload(this, Tag, Payload);
    ```

## Step 3: Listen for Messages

=== "Blueprint (Async Action)"
    1. In your Blueprint Event Graph, right-click and search **Relay Listen**.
    2. Set the **Channel Tag** and **Min Priority**.
    3. Connect the **On Message** output pin to your handler.

=== "Blueprint (Listener Group)"
    ```
    [BeginPlay] → [Create Relay Listener Group] → [Subscribe (Tag, Callback)]
    [EndPlay]   → [Listener Group > Unsubscribe All]
    ```

=== "C++"
    ```cpp
    #include "Relay/RelayListenerGroup.h"

    // Create in your actor
    ListenerGroup = NewObject<URelayListenerGroup>(this);

    ListenerGroup->Subscribe(
        FGameplayTag::RequestGameplayTag("Event.Combat"),
        FOnRelayMessage::CreateUObject(this, &AMyActor::HandleCombatEvent),
        /* MinPriority */ 0
    );

    // In destructor or EndPlay:
    ListenerGroup->UnsubscribeAll();
    ```

## Step 4: Sticky Channel

```cpp
// Mark a channel as sticky before broadcasting
URelayBlueprintLibrary::SetChannelSticky(this, FGameplayTag::RequestGameplayTag("GameState.Phase"), true);

// Broadcast the current phase — retained for late subscribers
URelayBlueprintLibrary::Broadcast(this, FGameplayTag::RequestGameplayTag("GameState.Phase.Combat"));
```

## Step 5: Message Filtering

```cpp
ListenerGroup->Subscribe(
    FGameplayTag::RequestGameplayTag("Event.Combat"),
    Callback,
    /* MinPriority */ 5,  // Only receive priority 5+
    /* ExcludeTag */ FGameplayTag::RequestGameplayTag("Event.Combat.Friendly") // Ignore friendly fire
);
```

## Step 6: Unsubscribe

```cpp
// Unsubscribe by handle
FRelaySubscriptionHandle Handle = ListenerGroup->Subscribe(Tag, Callback);
ListenerGroup->Unsubscribe(Handle);

// Or unsubscribe everything owned by the group
ListenerGroup->UnsubscribeAll();
```
