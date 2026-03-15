# Relay — Overview

## Architecture

Relay is a world-scoped message bus backed by a `UGameInstanceSubsystem`. All communication goes through `URelayBlueprintLibrary` static functions, making it accessible from anywhere without passing object references.

```mermaid
graph LR
    Publisher[Any Object] -->|Broadcast(Tag, Payload)| RelayBus[Relay Message Bus]
    RelayBus --> FilterEngine[Tag + Priority Filter]
    FilterEngine --> ListenerA[URelayListenerGroup A]
    FilterEngine --> ListenerB[URelayListenerGroup B]
    ListenerA -->|Delegate| HandlerA[Callback A]
    ListenerB -->|Delegate| HandlerB[Callback B]
```

## GameplayTag Routing

Every message carries a `FGameplayTag` channel. Listeners subscribe to exact tags or tag hierarchies. A listener subscribed to `Event.Combat` receives messages tagged `Event.Combat`, `Event.Combat.MeleeHit`, `Event.Combat.RangedHit`, etc. — standard GameplayTag parent matching.

## Priority

Messages carry an `int32` priority. Listeners can specify a minimum priority threshold, ignoring low-priority noise. When multiple listeners handle the same message they are invoked in priority-descending order.

## Sticky Channels

A "sticky" channel retains the last broadcast message. New listeners subscribing to a sticky channel immediately receive the most recent message. This is useful for state-like events (e.g., `GameState.Phase.Combat`) where latecomers need the current value.

## Listener Groups

`URelayListenerGroup` bundles multiple listener registrations under a single owner. Destroying the group (or calling `Unsubscribe`) removes all associated registrations atomically — no manual per-listener cleanup.

## Async Listen Action

`AsyncAction_RelayListen` is a Blueprint `UBlueprintAsyncActionBase` that exposes a latent **Listen** node. It waits for a message on a given tag, fires its output pin, then optionally repeats. This eliminates the need for a separate listener group object in simple Blueprint use cases.

## Performance

- Message dispatch is O(n) where n is the number of matching listeners for that tag.
- Listener lookup uses a `TMap<FGameplayTag, TArray<FRelayListenerEntry>>` — O(1) per channel.
- No heap allocation per dispatch — payloads are passed by const reference.
