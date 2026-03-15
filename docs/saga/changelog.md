# Saga — Changelog

## v1.0.0 — Initial Release

### Added
- `USagaSubsystem` — `UGameInstanceSubsystem` managing active, completed, and failed quests
- `USagaQuestData` — Data Asset defining quest metadata and task list
- `USagaTaskData` — Data Asset defining a single objective with required count
- Quest state machine: Not Started → Active → Completed / Failed
- Task state machine: Pending → In Progress → Completed / Failed
- `AdvanceTask` for count-based objective progress
- `CompleteTask` for instant task completion (trigger zones, dialogue, etc.)
- Optional task support — optional tasks don't block quest completion
- Delegates: `OnQuestStarted`, `OnQuestCompleted`, `OnQuestFailed`, `OnTaskAdvanced`, `OnTaskCompleted`
- Save/Load via `USagaSaveGame` for session persistence
- Full Blueprint-callable API on the subsystem
- Hidden quest type for secret/optional quest lines
- No GAS dependency
