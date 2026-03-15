# Lingua — Overview

## Architecture

```
ULinguaSubsystem  (UGameInstanceSubsystem)
    └── TMap<FName, FLinguaProficiency>  PlayerProficiencies

ULinguaLanguageData  (UDataAsset)
    ├── FName LanguageID
    ├── FText LanguageName
    ├── UFont* GlyphFont               — optional custom glyph font
    ├── TMap<FString, FString> GlyphMap — latin → glyph character mapping
    └── TArray<FLinguaVocabEntry> Vocabulary

FLinguaVocabEntry  (struct)
    ├── FString SourceWord
    ├── FString TranslatedWord
    └── float ProficiencyRequired      — 0.0–1.0 to reveal this word
```

## Language Data

Each language is a `ULinguaLanguageData` asset that defines:

- A unique `LanguageID` (e.g., `"Eldari"`, `"VoidScript"`)
- A custom glyph font (optional — if not provided, substitution characters are used)
- A character-level glyph mapping (e.g., `'a' → 'ᚠ'`)
- A vocabulary list with per-word proficiency reveal thresholds

## Proficiency System

The player's proficiency in each language is tracked as a float (0.0–1.0) per language. As proficiency increases:
- More vocabulary entries have their `ProficiencyRequired` threshold met
- Words above the threshold render as the translated word
- Words below the threshold render in the glyph script (or as `???`)

Proficiency is gained by:
- Reading language items in the world (`ULinguaSubsystem::AddProficiency`)
- Completing quests or finding lore collectibles
- Any game event — Lingua doesn't prescribe how proficiency is earned

## Text Rendering

`ULinguaSubsystem::RenderText(LanguageID, SourceText)` returns an `FText` ready for display. Words in the source text are matched against the vocabulary:

1. If the player's proficiency meets the word's threshold → returns translated word
2. If not → returns the glyph-mapped version (or `???` if no glyph map)

The result blends readable and unreadable text, giving the player a tangible sense of progress as they learn the language.
