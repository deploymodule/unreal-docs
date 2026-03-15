# Discourse — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Discourse"` to your `Build.cs`.

## 2. Author a Conversation (Data Asset)

1. Right-click → **Data Asset** → `UDiscourseConversationData`
2. Add lines via the Details panel:

| LineID | Speaker | Text | Choices |
|--------|---------|------|---------|
| `L001` | `Guard` | "Halt! Who goes there?" | Merchant / Run |
| `L002` | `Player` | "Just a humble trader." | — |
| `L003` | `Guard` | "Pass, then." | — |

## 3. Author via CSV Import

Create a DataTable with row type `FDiscourseLineData`:

```
Right-click → Miscellaneous → Data Table → FDiscourseLineData
File → Import → your_conversation.csv
```

## 4. Start a Conversation

```cpp
UDiscourseSubsystem* Discourse = GetGameInstance()->GetSubsystem<UDiscourseSubsystem>();
Discourse->StartConversation(GuardConversationData);
```

## 5. Drive UI via Delegates

```cpp
Discourse->OnLineReady.AddDynamic(this, &AMyUI::OnLineReady);
Discourse->OnChoiceReady.AddDynamic(this, &AMyUI::OnChoiceReady);
Discourse->OnConversationEnded.AddDynamic(this, &AMyUI::OnConversationEnded);

void AMyUI::OnLineReady(const FDiscourseLineData& Line, UDiscourseSpeakerData* Speaker)
{
    DialogueTextWidget->SetText(Line.LineText);
    SpeakerNameWidget->SetText(Speaker->DisplayName);
    PortraitWidget->SetBrushFromTexture(Speaker->Portrait);
}
```

## 6. Advance the Conversation

Call `Advance()` when the player presses the continue button:

```cpp
Discourse->Advance();
```

## 7. Make a Choice

```cpp
void AMyUI::OnChoiceReady(const TArray<FDiscourseChoice>& Choices)
{
    for (int32 i = 0; i < Choices.Num(); ++i)
    {
        ChoiceButtons[i]->SetText(Choices[i].ChoiceText);
        ChoiceButtons[i]->OnClicked.AddDynamic([this, i]() {
            Discourse->MakeChoice(i);
        });
    }
}
```

## 8. Queue a Conversation

```cpp
// This will play after the current conversation ends
Discourse->QueueConversation(FollowUpConversationData);
```

## 9. Skip / End Immediately

```cpp
Discourse->SkipToEnd();
```
