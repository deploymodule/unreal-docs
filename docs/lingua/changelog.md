# Lingua — Changelog

## v1.0.0 — Initial Release

### Added
- `ULinguaSubsystem` — `UGameInstanceSubsystem` managing per-language player proficiencies
- `ULinguaLanguageData` — Data Asset defining a language's vocabulary and glyph mapping
- `FLinguaVocabEntry` — per-word translation entries with proficiency reveal thresholds
- `RenderText` function for mixed-proficiency text display (translated + glyph words)
- Character-level glyph substitution map support (latin → custom glyph characters)
- Optional custom glyph `UFont` per language
- `AddProficiency` and `SetProficiency` for flexible proficiency progression
- `KnowsWord` per-word proficiency query
- `GetKnownWords` for building language journals or vocabulary UI
- Delegates: `OnProficiencyChanged`, `OnWordUnlocked`
- Save/Load proficiency persistence
- Dynamic language registration via `RegisterLanguage`
- Blueprint-callable API for all subsystem functions
- Client-side only — no replication (proficiency is per-player local state)
