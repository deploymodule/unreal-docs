# Lingua — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Lingua"` to your `Build.cs`.

## 2. Create a Language Data Asset

1. Right-click → **Data Asset** → `ULinguaLanguageData`
2. Configure:

| Field | Value |
|-------|-------|
| Language ID | `Eldari` |
| Language Name | `The Elder Script` |
| Glyph Font | Assign your rune font (optional) |

3. Add Vocabulary entries:

| Source Word | Translation | Proficiency Required |
|-------------|-------------|----------------------|
| `danger` | `varas` | 0.1 |
| `door` | `melden` | 0.0 |
| `ancient` | `solen` | 0.3 |
| `magic` | `aethari` | 0.5 |

## 3. Display Text in the Language

```cpp
ULinguaSubsystem* Lingua = GetGameInstance()->GetSubsystem<ULinguaSubsystem>();

// Raw text in "Eldari" (source language)
FString SourceText = "The ancient door holds magic danger";

// Render based on current proficiency
FText DisplayText = Lingua->RenderText(FName("Eldari"), SourceText);

MyTextWidget->SetText(DisplayText);
```

At zero proficiency, all words render as glyphs. At full proficiency, all words render as translations.

## 4. Grant Proficiency

```cpp
// Player reads a rune tablet — grant 0.1 proficiency
Lingua->AddProficiency(FName("Eldari"), 0.1f);

// Set proficiency directly
Lingua->SetProficiency(FName("Eldari"), 0.5f);
```

## 5. Query Proficiency

```cpp
float Proficiency = Lingua->GetProficiency(FName("Eldari"));
bool bKnowsWord   = Lingua->KnowsWord(FName("Eldari"), "magic");
```

## 6. Bind to Proficiency Changes

```cpp
Lingua->OnProficiencyChanged.AddDynamic(this, &AMyChar::OnLinguaProficiencyChanged);

void AMyChar::OnLinguaProficiencyChanged(FName LanguageID, float OldValue, float NewValue)
{
    // Update journal UI, show "You understand more Eldari" notification, etc.
}
```

## 7. Save and Load Proficiency

```cpp
Lingua->Save();   // Persists all language proficiencies
Lingua->Load();
```
