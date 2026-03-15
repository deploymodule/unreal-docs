# Saga — API Reference

## USagaSubsystem

`#include "SagaSubsystem.h"`

### Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `StartQuest` | `bool StartQuest(USagaQuestData* Quest)` | Begin tracking a quest |
| `FailQuest` | `void FailQuest(FName QuestID)` | Mark a quest failed |
| `AbandonQuest` | `void AbandonQuest(FName QuestID)` | Remove quest from active list |
| `AdvanceTask` | `void AdvanceTask(FName QuestID, FName TaskID, int32 Amount)` | Add progress to a task |
| `CompleteTask` | `void CompleteTask(FName QuestID, FName TaskID)` | Force-complete a task |
| `IsQuestActive` | `bool IsQuestActive(FName QuestID)` | Check active state |
| `IsQuestCompleted` | `bool IsQuestCompleted(FName QuestID)` | Check completed state |
| `IsQuestFailed` | `bool IsQuestFailed(FName QuestID)` | Check failed state |
| `GetTaskCount` | `int32 GetTaskCount(FName QuestID, FName TaskID)` | Current task progress |
| `GetActiveQuests` | `TArray<USagaQuestData*> GetActiveQuests()` | All currently active quests |
| `Save` | `void Save()` | Write quest state to save game |
| `Load` | `void Load()` | Read quest state from save game |

### Delegates

| Delegate | Signature | Fired when |
|----------|-----------|------------|
| `OnQuestStarted` | `(FName QuestID, USagaQuestData*)` | Quest becomes active |
| `OnQuestCompleted` | `(FName QuestID, USagaQuestData*)` | All required tasks complete |
| `OnQuestFailed` | `(FName QuestID, USagaQuestData*)` | Quest is failed |
| `OnTaskAdvanced` | `(FName QuestID, FName TaskID, int32 Current, int32 Required)` | Task progress updated |
| `OnTaskCompleted` | `(FName QuestID, FName TaskID)` | Task reaches required count |

## USagaQuestData

| Property | Type | Description |
|----------|------|-------------|
| `QuestID` | FName | Unique quest identifier |
| `Title` | FText | Display title |
| `Description` | FText | Narrative description |
| `QuestType` | ESagaQuestType | Main / Side / Hidden |
| `Tasks` | TArray\<USagaTaskData*\> | Ordered task list |

## USagaTaskData

| Property | Type | Description |
|----------|------|-------------|
| `TaskID` | FName | Unique task identifier |
| `Description` | FText | Task description shown in UI |
| `RequiredCount` | int32 | Count to reach for completion |
| `bOptional` | bool | Optional tasks don't block quest completion |
