# Saga — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Saga"` to your `Build.cs`.

## 2. Create a Task Data Asset

1. Right-click in Content Browser → **Data Asset** → `USagaTaskData`
2. Configure:

| Field | Value |
|-------|-------|
| Task ID | `Task_KillWolves` |
| Description | `Eliminate 5 wolves in the forest` |
| Required Count | `5` |
| Optional | `false` |

## 3. Create a Quest Data Asset

1. Right-click → **Data Asset** → `USagaQuestData`
2. Configure:

| Field | Value |
|-------|-------|
| Quest ID | `Quest_ForestClearance` |
| Title | `Clear the Forest` |
| Description | `The village needs the forest cleared of wolves` |
| Type | `Side` |
| Tasks | `[Task_KillWolves, Task_ReportToVillager]` |

## 4. Start a Quest

```cpp
USagaSubsystem* Saga = GetGameInstance()->GetSubsystem<USagaSubsystem>();
Saga->StartQuest(ForestClearanceQuestData);
```

## 5. Advance Task Progress

Call this whenever a relevant game event occurs (enemy killed, item collected, etc.):

```cpp
// Wolf was killed — advance the kill task by 1
Saga->AdvanceTask(FName("Quest_ForestClearance"), FName("Task_KillWolves"), 1);
```

## 6. Respond to Delegate Events

```cpp
Saga->OnTaskCompleted.AddDynamic(this, &AMyGameMode::OnTaskCompleted);
Saga->OnQuestCompleted.AddDynamic(this, &AMyGameMode::OnQuestCompleted);

void AMyGameMode::OnQuestCompleted(FName QuestID, USagaQuestData* QuestData)
{
    // Award XP, unlock next quest, play fanfare, etc.
}
```

## 7. Query Quest State

```cpp
bool bActive    = Saga->IsQuestActive(FName("Quest_ForestClearance"));
bool bComplete  = Saga->IsQuestCompleted(FName("Quest_ForestClearance"));
int32 Progress  = Saga->GetTaskCount(FName("Quest_ForestClearance"), FName("Task_KillWolves"));
```

## 8. Fail a Quest

```cpp
Saga->FailQuest(FName("Quest_ForestClearance"));
```

## 9. Save and Load

```cpp
Saga->Save();   // Persist quest state to save game
Saga->Load();   // Restore quest state from save game
```
