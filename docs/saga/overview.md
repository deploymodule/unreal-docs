# Saga — Overview

## Architecture

```
USagaSubsystem  (UGameInstanceSubsystem)
    ├── TArray<FSagaActiveQuest>   ActiveQuests
    ├── TArray<FName>              CompletedQuestIDs
    └── TArray<FName>              FailedQuestIDs

USagaQuestData  (UDataAsset)
    ├── FName QuestID
    ├── FText Title / Description
    ├── TArray<USagaTaskData*> Tasks
    └── ESagaQuestType (Main / Side / Hidden)

USagaTaskData  (UDataAsset)
    ├── FName TaskID
    ├── FText Description
    ├── int32 RequiredCount      — for countable tasks (collect 5, kill 10)
    └── bool bOptional
```

## Quest Hierarchy

Saga uses a two-level hierarchy:

- **Quest** — the top-level story unit. Has a title, description, and an ordered list of tasks.
- **Task** — a single objective within a quest (go to location, kill N enemies, collect N items, talk to NPC, etc.).

Tasks do not know *how* they are completed — that logic lives in your game code. Saga only tracks progress counts and completion state.

## State Machine

Each active quest goes through these states:

```
Not Started → Active → [All required tasks complete] → Completed
                     → [Fail condition triggered]   → Failed
```

Individual tasks transition:
```
Pending → In Progress → Completed
                     → Failed (optional tasks only)
```

## Data Flow

Game code calls `USagaSubsystem::AdvanceTask(QuestID, TaskID, Amount)` to report progress. The subsystem:

1. Finds the active quest
2. Updates the task's current count
3. If `CurrentCount >= RequiredCount`, marks the task complete
4. If all required tasks are complete, marks the quest complete and fires `OnQuestCompleted`
5. Fires `OnTaskAdvanced` and `OnTaskCompleted` delegates at each step

## Persistence

Quest state is serialized to a `USagaSaveGame` object. Call `USagaSubsystem::Save()` and `Load()` to persist across sessions.
