# --team Mode (Agent Teams)

Use `--team` flag for large features requiring parallel implementation/review.

**Cost:** 3-5x tokens vs standard mode. Use for complex features only.

## When to Use

Use --team:
- Feature with 4+ files across folders
- Complex data flow needing parallel work
- Time-sensitive large features
- Critical feature requiring multi-perspective review

Skip --team:
- Single component/hook/utility
- Small bug fixes
- Features touching 1-2 files
- Quick iteration cycles

## Coordination Protocol

1. **Shared Task List**: All agents use TaskCreate/TaskUpdate for progress
2. **Interface Changes**: When types/state change, notify other agents via messages
3. **No File Conflicts**: Each agent owns distinct folders
4. **Integration Point**: Main agent coordinates final integration (barrel exports, module config)

## Workflow

```
1. Main agent creates Task List from Implementation Checklist
2. Spawn N parallel agents based on feature scope
3. Each agent:
   - Claims tasks via TaskUpdate (owner, in_progress)
   - Implements assigned files
   - Marks tasks completed
   - Notifies if interface changes affect others
4. Main agent:
   - Monitors progress via TaskList
   - Handles integration
   - Runs verification commands
```
