# Discourse — API Reference

## UDiscourseSubsystem

`#include "DiscourseSubsystem.h"`

### Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `StartConversation` | `bool StartConversation(UDiscourseConversationData* Data)` | Begin a conversation immediately |
| `QueueConversation` | `void QueueConversation(UDiscourseConversationData* Data)` | Add to conversation queue |
| `Advance` | `void Advance()` | Move to the next line |
| `MakeChoice` | `void MakeChoice(int32 ChoiceIndex)` | Select a choice by index |
| `SkipToEnd` | `void SkipToEnd()` | Immediately end the active conversation |
| `IsConversationActive` | `bool IsConversationActive()` | Check if a conversation is playing |
| `GetCurrentLine` | `FDiscourseLineData GetCurrentLine()` | Current line data |
| `GetActiveSpeaker` | `UDiscourseSpeakerData* GetActiveSpeaker()` | Current speaker data asset |

### Delegates

| Delegate | Signature | Fired when |
|----------|-----------|------------|
| `OnConversationStarted` | `(UDiscourseConversationData*)` | Conversation begins |
| `OnLineReady` | `(FDiscourseLineData, UDiscourseSpeakerData*)` | New line ready to display |
| `OnChoiceReady` | `(TArray<FDiscourseChoice>)` | Player must select a choice |
| `OnConversationEnded` | `(UDiscourseConversationData*)` | Conversation completes |

## FDiscourseLineData

| Field | Type | Description |
|-------|------|-------------|
| `LineID` | FName | Unique line identifier |
| `SpeakerID` | FName | Speaker reference |
| `LineText` | FText | Localized dialogue text |
| `Conditions` | TArray\<FDiscourseCondition\> | Ledger conditions to pass |
| `Choices` | TArray\<FDiscourseChoice\> | Choice options (empty = no choice) |
| `NextLineID` | FName | Explicit next line (empty = sequential) |

## FDiscourseChoice

| Field | Type | Description |
|-------|------|-------------|
| `ChoiceText` | FText | Display text for this option |
| `NextLineID` | FName | Line to jump to when selected |
| `Conditions` | TArray\<FDiscourseCondition\> | Conditions to show this choice |

## UDiscourseSpeakerData

| Property | Type | Description |
|----------|------|-------------|
| `SpeakerID` | FName | Unique speaker key |
| `DisplayName` | FText | Name shown in UI |
| `Portrait` | UTexture2D* | Speaker portrait texture |
| `AudioBank` | USoundBase* | Voice line bank reference |
