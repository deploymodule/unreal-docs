# Ledger — Changelog

## v1.0.0 — Initial Release

### Added
- `ULedgerSubsystem` — `UGameInstanceSubsystem` providing a global fact key-value store
- Five fact types: Bool, Int, Float, Name, String
- `ULedgerGateComponent` — actor component evaluating fact conditions to open/close gates
- Gate logic modes: All (AND) and Any (OR) condition evaluation
- Six comparison operators: Equal, NotEqual, GreaterThan, LessThan, GreaterEqual, LessEqual
- Reactive gate evaluation — conditions re-evaluated automatically on any fact change
- `OnFactChanged` delegate on subsystem for reactive programming patterns
- `OnGateOpened` / `OnGateClosed` delegates on gate component
- `ULedgerBFL` Blueprint function library wrapping all subsystem calls as static nodes
- Save/Load persistence via `SaveToSlot` / `LoadFromSlot`
- Server-authoritative writes with client-side replication on change
- Facts persist across level transitions (stored in Game Instance subsystem)
- No GAS dependency
