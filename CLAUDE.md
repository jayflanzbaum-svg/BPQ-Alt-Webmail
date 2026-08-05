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

## Release checklist (keep everything in sync)

There is no single command for this — a release touches the git branch, a tag, and a
GitHub Release object, which are three separate systems. When asked to "release vX.Y.Z",
do all of the following, in order:

1. **Bump the version string in `bpq-alt-webmail.html`** — it appears in exactly 3 places
   (grep for the previous version to find them): the topbar `<span>`, the sidebar-footer
   `#si-ver` div, and the mobile settings `info.innerHTML` line.
2. **Add a `## vX.Y.Z — YYYY-MM-DD` entry to `CHANGELOG.md`** (top of file, above the
   previous entry) describing what changed.
3. **Update `STATUS.md`** per the Obsidian-sync rule above.
4. **Commit** the above on `main` — day-to-day work happens directly on `main`
   (consolidated 2026-08-05; the old `experimental/jay-dev` branch is gone). What counts
   as "released" is defined by tags and GitHub Releases, not a branch. Use a short-lived
   feature branch only for genuinely experimental work, and merge it when done.
5. **Push**: `git push origin main`.
6. **Tag `main`**: `git tag -a vX.Y.Z -m "vX.Y.Z -- <short summary>"` then `git push origin vX.Y.Z`.
   A tag alone does NOT create a GitHub Release or update "Latest release" — step 7 does that.
7. **Publish the GitHub Release** for that tag with the `gh` CLI (installed 2026-07-19), e.g.:
   `gh release create vX.Y.Z bpq-alt-webmail.html --repo jayflanzbaum-svg/BPQ-Alt-Webmail --title "vX.Y.Z -- <short summary>" --notes-file <changelog-excerpt>`
   This tags-if-needed, publishes the release notes, and attaches `bpq-alt-webmail.html` as a
   downloadable asset all in one call. A tag alone does NOT create a GitHub Release or update
   "Latest release" — this step is what does that.

   **Auth gotcha**: `gh auth login --with-token` fails with `401 Bad credentials` against the
   fine-grained PAT stored in Git Credential Manager, even though the token itself is valid
   (confirmed working directly against the REST API) — this is a known gh limitation validating
   fine-grained tokens, not a bad token. Workaround: skip `gh auth login` entirely and set the
   `GH_TOKEN` env var for the command instead, pulling the token from Git Credential Manager
   non-interactively (PATH must include Git's cmd dir first in this shell):
   ```powershell
   $env:PATH = "C:\Program Files\Git\cmd;C:\Program Files\Git\clangarm64\bin;C:\Program Files\GitHub CLI;" + $env:PATH
   $env:GH_TOKEN = ("protocol=https`nhost=github.com`n`n" | git-credential-manager.exe get | Select-String "^password=").ToString().Substring(9)
   gh release create vX.Y.Z bpq-alt-webmail.html --repo jayflanzbaum-svg/BPQ-Alt-Webmail --title "..." --notes-file "..."
   ```
   Both `$env:PATH` and `$env:GH_TOKEN` must be set in the same command block that calls `gh`,
   since shell state does not persist between separate command invocations.
8. **Verify**: `git status` clean, `git log --oneline -1 main` matches `origin/main`,
   and `gh release list --repo jayflanzbaum-svg/BPQ-Alt-Webmail` shows the new tag
   marked `Latest`.
