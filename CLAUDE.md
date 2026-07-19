<!-- STATUS-RULE v1 -- seeded by the Obsidian vault refresh (projects/refresh-projects.ps1). Edit the prose freely; keep this marker line so it is not re-appended. -->
## Project status log (Obsidian sync)

Maintain a file named `STATUS.md` at the repo root. Update it whenever we finish a
meaningful chunk of work: a feature, a fix, a decision, or the end of a session.
Keep it SHORT and CURRENT -- overwrite stale lines, trim "Recent work" to the last
~5 entries. Use exactly this structure and these heading names (Jason's Obsidian
vault parses them and surfaces them on his project dashboard):

```markdown
---
updated: YYYY-MM-DD
summary: One sentence on what this project is right now.
---

## Recent work
- YYYY-MM-DD: one line on what changed

## Open issues
- [ ] Things that still need doing

## Future ideas
- [ ] Not-yet-committed ideas

## Decisions & blockers
- Short note on why things are the way they are, or what is stuck
```

Use today's real date in ISO format. This file is a hand-off summary for an external
knowledge vault, not a changelog: keep it current, not append-only.
