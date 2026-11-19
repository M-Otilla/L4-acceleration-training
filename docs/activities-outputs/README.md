# Activity outputs

Evidence of your work for the weekly reviews. Commit it with the work it belongs to.

| Folder | What goes in it |
|---|---|
| `session-logs/` | One log per Claude Code session, written automatically by the session-log hook. Copilot SDK session snapshots may also be recorded here from local session telemetry; those are explicitly labeled and are not updated by the Claude Code hook |
| `usage.md` | One line per model run in a session (a new line each time the model changes), with its last activity time, total tokens, model and estimated cost, written automatically by the session-log hook for Claude Code sessions. Copilot SDK snapshots are added from local telemetry; cost is `n/a` when unavailable. Commit it with the `session-logs/` files |
| `screenshots/` | Screenshots the weekly deliverables ask for, named by week and subject, for example `week-1-f2-spec.png` or `week-3-review-not-ok.png` |
