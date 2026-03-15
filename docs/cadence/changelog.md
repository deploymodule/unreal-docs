# Cadence — Changelog

## v1.0.0 — Initial Release

### Added
- `UCadenceComponent` — rolling input history with configurable time window
- `UCadenceComboData` — Data Asset defining ordered input sequences
- Subsequence matching mode (extra inputs allowed between steps)
- Exact matching mode (`bRequireExact = true`, no extra inputs permitted)
- Priority-based conflict resolution when multiple combos match simultaneously
- Prefix combo delay (`r.Cadence.PrefixDelay`) to allow longer combos to supersede shorter ones
- Per-combo cooldown with `IsComboReady` query
- Partial progress query via `GetPartialProgress` (0–1 fraction of sequence completed)
- `OnComboMatched` dynamic multicast delegate
- Full Blueprint-callable API
- Debug overlay for input history and partial match state (`r.Cadence.Debug 1`)
- No GAS dependency, no Enhanced Input coupling — works with any input backend
