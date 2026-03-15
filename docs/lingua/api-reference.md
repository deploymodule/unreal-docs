# Lingua — API Reference

## ULinguaSubsystem

`#include "LinguaSubsystem.h"`

### Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `RenderText` | `FText RenderText(FName LanguageID, const FString& SourceText)` | Render source text at current proficiency |
| `AddProficiency` | `void AddProficiency(FName LanguageID, float Amount)` | Increase proficiency |
| `SetProficiency` | `void SetProficiency(FName LanguageID, float Value)` | Set proficiency directly (0–1) |
| `GetProficiency` | `float GetProficiency(FName LanguageID)` | Current proficiency (0–1) |
| `KnowsWord` | `bool KnowsWord(FName LanguageID, const FString& Word)` | Check if proficiency reveals a word |
| `GetKnownWords` | `TArray<FString> GetKnownWords(FName LanguageID)` | All currently readable words |
| `RegisterLanguage` | `void RegisterLanguage(ULinguaLanguageData* Language)` | Register a language data asset |
| `GetLanguage` | `ULinguaLanguageData* GetLanguage(FName LanguageID)` | Retrieve language data |
| `GetAllLanguageIDs` | `TArray<FName> GetAllLanguageIDs()` | All registered language IDs |
| `Save` | `void Save()` | Persist proficiencies to save game |
| `Load` | `void Load()` | Restore proficiencies from save game |

### Delegates

| Delegate | Signature | Fired when |
|----------|-----------|------------|
| `OnProficiencyChanged` | `(FName LanguageID, float OldValue, float NewValue)` | Proficiency increases |
| `OnWordUnlocked` | `(FName LanguageID, FString Word)` | A word's threshold is newly met |

## ULinguaLanguageData

`#include "LinguaLanguageData.h"`

| Property | Type | Description |
|----------|------|-------------|
| `LanguageID` | FName | Unique language identifier |
| `LanguageName` | FText | Display name |
| `GlyphFont` | UFont* | Optional custom glyph font |
| `GlyphMap` | TMap\<FString, FString\> | Character substitution map |
| `Vocabulary` | TArray\<FLinguaVocabEntry\> | Word-level translation entries |

## FLinguaVocabEntry

| Field | Type | Description |
|-------|------|-------------|
| `SourceWord` | FString | Word in the source language |
| `TranslatedWord` | FString | Revealed translation |
| `ProficiencyRequired` | float | 0–1 proficiency to reveal this word |
