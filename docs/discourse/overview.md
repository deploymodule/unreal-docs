# Discourse — Overview

## Architecture

```
UDiscourseSubsystem  (UGameInstanceSubsystem)
    ├── UDiscourseConversationData*  ActiveConversation
    ├── TQueue<UDiscourseConversationData*>  ConversationQueue
    └── FDiscourseStage  CurrentStage

UDiscourseConversationData  (UDataAsset or DataTable row)
    └── TArray<FDiscourseLineData>  Lines

FDiscourseLineData  (struct / DataTable row)
    ├── FName LineID
    ├── FName SpeakerID
    ├── FText LineText
    ├── TArray<FDiscourseCondition> Conditions
    ├── TArray<FDiscourseChoice>   Choices     — if non-empty, this line is a choice node
    ├── FName NextLineID                        — explicit jump (empty = sequential)
    └── FName ConversationEndAction             — "End", "QueueNext", custom tag
```

## CSV Import

Conversations can be authored in a spreadsheet and imported as a `UDataTable` with row struct `FDiscourseLineData`. Column headers map directly to struct fields.

Example CSV:

```csv
LineID,SpeakerID,LineText,Conditions,Choices,NextLineID
001,NPC_Guard,"Halt! Who goes there?","","[{ChoiceText:'I'm a merchant',NextLineID:'002'},{ChoiceText:'Run!',NextLineID:'003'}]",""
002,Player,"Just a humble trader passing through.","","","004"
003,NPC_Guard,"Stop right there!","","","END"
```

## Staging System

The staging system controls conversation flow:

1. **Start** — Conversation begins; first line is evaluated (conditions checked)
2. **Line** — Current line is broadcast via `OnLineReady`; UI displays it
3. **Advance** — Caller calls `Advance()` (e.g., player presses continue)
4. **Choice** — If line has choices, `OnChoiceReady` fires; UI shows options. Player picks one.
5. **Jump** — Next line ID resolved (explicit jump, choice-driven, or sequential)
6. **End** — Conversation ends; `OnConversationEnded` fires; next queued conversation starts

## Condition System

Lines can have conditions that must pass for the line to play. Conditions query the Ledger subsystem:

```
Condition: "BossDefeated == true"
```

If conditions fail, the line is skipped and the next line is evaluated.

## Speaker System

Speaker IDs map to `UDiscourseSpeakerData` assets containing display name, portrait, and audio bank. Discourse broadcasts speaker data with each line event so UI can update portraits without coupling.
