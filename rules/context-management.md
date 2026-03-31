# Context Management

Rules to prevent context pollution in long sessions.

## Session Isolation
- Separate sessions by feature. Don't do multiple features in one session.
- Each session focuses on one feature/task.

## Compaction
- Run `/compact` immediately when Claude starts ignoring CLAUDE.md rules
- Project context is maintained even after compaction

## Subagent Delegation
- (see claude-philosophy.md #3: Separate Context with Subagents)
- `context: fork` skills: explore, cso, review, plan-ceo-review

## Parallel Development with Git Worktree
Agent Teams are enabled, so parallel development with worktrees is possible:
```bash
# Terminal 1: feature A
git worktree add ../project-auth feature/auth
cd ../project-auth && claude

# Terminal 2: feature B (concurrent)
git worktree add ../project-ws feature/websocket
cd ../project-ws && claude
```
- Each session uses separate git checkout, no file conflicts
- Run long tasks in background with `claude -p "..." &`
- Notification hook alerts on completion
