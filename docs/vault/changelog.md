# Vault — Changelog

## v1.0.0 — Initial Release

### Added
- `UVaultContainerComponent` with slot count, weight limit, and optional rule set validation.
- `UVaultSubsystem` with `AddItem`, `RemoveItem`, `TransferItem`, `HydrateItem`, `GetTotalCount`, `GetEffectiveStats`.
- `UVaultItemDefinition` Data Asset with display name, weight, stack size, tags, and soft mesh reference.
- `FVaultItemEntry` minimal storage struct with item ID, count, modifiers, and free-form metadata.
- `UVaultRuleSet` Data Asset with `CanAdd`, `CanRemove`, `CanTransfer` rule evaluation.
- `FVaultModifier` struct for enchantment/condition/buff stacking with lazy stat resolution.
- Transaction log via `FVaultTransaction` — full add/remove/transfer history this session.
- Replication via `FFastArraySerializer` — only changed entries replicated.
- Courier integration for on-demand `UVaultItemDefinition` loading.
- `OnItemAdded` and `OnItemRemoved` delegates on `UVaultContainerComponent`.
- `EVaultResult` enum for rich failure diagnostics.
- Full Blueprint exposure for all subsystem and component functions.
