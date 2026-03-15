# Relay — Changelog

## v1.0.0 — Initial Release

### Added
- `URelayBlueprintLibrary` with `Broadcast`, `BroadcastWithPayload`, sticky channel APIs.
- `URelayListenerGroup` with `Subscribe`, `Unsubscribe`, `UnsubscribeAll`, and `SetPaused`.
- GameplayTag hierarchy matching — listeners on parent tags receive child tag messages.
- Priority dispatch: listeners invoked in descending priority order; `MinPriority` filter support.
- Sticky channel support: `SetChannelSticky`, `ClearStickyChannel`, `GetLastStickyMessage`.
- `ExcludeTag` filter on subscriptions for fine-grained message filtering.
- `AsyncAction_RelayListen` Blueprint latent node with Repeat and Timeout options.
- `FRelayMessage` struct with channel, priority, sender, and `FInstancedStruct` payload.
- `FRelaySubscriptionHandle` opaque handle for targeted unsubscription.
- Full Blueprint exposure for all library and group functions.
- O(1) channel lookup via `TMap<FGameplayTag, TArray<FRelayListenerEntry>>`.
