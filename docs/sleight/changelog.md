# Sleight — Changelog

## v1.0.0 — Initial Release

### Added
- `USleightGameStateComponent` with full game session lifecycle: initialize, deal hands, play cards, resolve stack, combat.
- `USleightCardData` Data Asset with cost, keywords, effects (FSleightEffect), attack/defense, and soft artwork reference.
- `SleightEffectStack` with ordered push/pop resolution and interject support.
- `SleightCombatResolver` with Declare Attackers → Declare Blockers → Deal Damage → Apply Effects phases.
- Five-zone system: Deck, Hand, Battlefield, Graveyard, Exile.
- `USleightRules_Deckbuilder` and `USleightRules_TCG` base rule classes for subclassing.
- Keyword registry with `RegisterKeyword` API for custom keyword handlers.
- Replication via `USleightGameStateComponent` on the Game State actor.
- `OnCardPlayed`, `OnEffectResolved`, `OnZoneChanged`, `OnCombatPhaseChanged`, `OnGameOver` delegates.
- Full Blueprint exposure for all component and data functions.
