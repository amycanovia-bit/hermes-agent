# Action-item template

Use this structure for planning items in .plans and docs/plans so follow-up is consistent.

| Field | Required | Notes |
|------|----------|-------|
| Status | yes | One of: planned, in_progress, blocked, done |
| Code area | yes | Primary file, module, or subsystem |
| Supporting docs | yes | Planning doc(s), design note(s), or reference docs |
| Creator | yes | Person who opened/owns the item, or `unassigned` |
| Action item | yes | Short imperative description of the work |
| Notes | optional | Constraints, links, or implementation context |

Example:

| Status | Code area | Supporting docs | Creator | Action item | Notes |
|-------|-----------|-----------------|---------|-------------|-------|
| planned | gateway/run.py | .plans/streaming-support.md | unassigned | Add stream callback plumbing | Keep non-streaming path as default |
