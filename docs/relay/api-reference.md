# Relay — API Reference

## URelayBlueprintLibrary

Static library — call from anywhere.

| Function | Signature | Description |
|---|---|---|
| `Broadcast` | `void (UObject* Context, FGameplayTag Channel, int32 Priority)` | Broadcasts a message on the given channel with no payload. |
| `BroadcastWithPayload` | `void (UObject* Context, FGameplayTag Channel, const FRelayPayload& Payload, int32 Priority)` | Broadcasts a message with a typed payload struct. |
| `SetChannelSticky` | `void (UObject* Context, FGameplayTag Channel, bool bSticky)` | Marks a channel as sticky (retains last broadcast for late subscribers). |
| `ClearStickyChannel` | `void (UObject* Context, FGameplayTag Channel)` | Removes the retained message from a sticky channel. |
| `GetLastStickyMessage` | `bool (UObject* Context, FGameplayTag Channel, FRelayMessage& OutMessage)` | Returns the last message on a sticky channel. Returns false if none. |

## URelayListenerGroup

| Function | Signature | Description |
|---|---|---|
| `Subscribe` | `FRelaySubscriptionHandle (FGameplayTag, FOnRelayMessage, int32 MinPriority, FGameplayTag ExcludeTag)` | Registers a listener for the given channel. Returns a handle for individual unsubscription. |
| `Unsubscribe` | `void (FRelaySubscriptionHandle)` | Removes the listener associated with the given handle. |
| `UnsubscribeAll` | `void ()` | Removes all listeners owned by this group. |
| `SetPaused` | `void (bool)` | Pauses or resumes all listeners in this group without unsubscribing. |
| `GetSubscriptionCount` | `int32 ()` | Returns the number of active subscriptions. |

## AsyncAction_RelayListen

Blueprint async action node for latent message listening.

| Pin | Description |
|---|---|
| **Channel** | The GameplayTag to listen on. |
| **Min Priority** | Minimum message priority to receive. |
| **Repeat** | If true the action fires repeatedly on each message; if false it fires once and completes. |
| **On Message** (output) | Fires when a matching message is received. Carries `FRelayMessage`. |
| **On Timeout** (output) | Fires if `Timeout` seconds elapse with no message (0 = no timeout). |

## FRelayMessage (Struct)

| Field | Type | Description |
|---|---|---|
| `Channel` | `FGameplayTag` | The channel this message was broadcast on. |
| `Priority` | `int32` | Priority of the broadcast. |
| `Sender` | `TWeakObjectPtr<UObject>` | The object that called Broadcast. |
| `Payload` | `FInstancedStruct` | Optional typed payload. Use `GetPayload<T>()` to extract. |

## FRelaySubscriptionHandle

Opaque handle returned by `Subscribe`. Pass to `Unsubscribe` for targeted removal. Invalidates automatically when the owning `URelayListenerGroup` is destroyed.
