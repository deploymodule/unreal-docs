# Discourse — Changelog

## v1.0.0 — Initial Release

### Added
- `UDiscourseSubsystem` — `UGameInstanceSubsystem` managing conversation staging and queuing
- `UDiscourseConversationData` — Data Asset holding an ordered list of dialogue lines
- `FDiscourseLineData` — DataTable-compatible struct for CSV-importable dialogue authoring
- CSV import workflow via `UDataTable` with `FDiscourseLineData` row struct
- Linear conversation flow with explicit `NextLineID` jump support
- Choice nodes with per-choice next-line routing and optional display conditions
- Condition-gated line skipping via Ledger subsystem integration
- Sequential line fallback (empty `NextLineID` advances to next array entry)
- Speaker system with `UDiscourseSpeakerData` assets (display name, portrait, audio bank)
- Conversation queue: multiple conversations queue up and play in order
- Delegates: `OnConversationStarted`, `OnLineReady`, `OnChoiceReady`, `OnConversationEnded`
- `SkipToEnd` for ESC-style dialogue skip
- Blueprint-callable API for all subsystem functions
- No UI dependency — fully delegate-driven, works with any UI system
