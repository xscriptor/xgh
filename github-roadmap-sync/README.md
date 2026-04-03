# Roadmap Sync — Usage Guide

A system that keeps your `ROADMAP.md` in sync with GitHub Issues automatically using a Python script and a GitHub Actions workflow.

## Overview

### Scripts

| File | Description |
|------|-------------|
| `.github/scripts/sync_roadmap.py` | Python script that parses `ROADMAP.md`, synchronizes checkbox states with GitHub Issues, auto-creates missing issues, and commits changes back. |
| `.github/workflows/roadmap-sync.yml` | GitHub Actions workflow that triggers the sync script on pushes to `main` or via manual dispatch. |

### How It Works

Your `ROADMAP.md` is the **single source of truth**. Edit checkboxes, push to `main`,
and the GitHub Action handles the rest:

| Checkbox | Effect on GitHub Issue |
|----------|----------------------|
| `- [ ]`  | Open (or reopen if closed); remove `in-progress` label if present |
| `- [/]`  | Open (or reopen if closed); add `in-progress` label |
| `- [x]`  | Close issue; remove `in-progress` label if present |
| No `(#N)` | **Auto-creates** the issue, writes the number back to the file, and commits |
| Modify text | **Renames** the GitHub issue to match the updated text in the roadmap |

## Script Details — `sync_roadmap.py`

The sync engine runs in two phases:

### Phase 1 — Auto-create issues

Tasks in the roadmap that don't have an issue number (`(#N)`) are processed first:

- A search is performed to **avoid duplicates** — if an issue with the exact same title already exists, it is reused instead of creating a new one.
- New issues are created via `gh issue create` with the phase label applied automatically.
- Tasks marked `[x]` without an issue are **created and immediately closed**.
- After creation, the script **updates the ROADMAP.md** in-place with the assigned issue number and **auto-commits and pushes** using the bot identity `roadmap-sync[bot]`.
- The commit message includes `[roadmap-sync]` to prevent infinite workflow loops.

### Phase 2 — Sync state

All tasks with issue numbers are synced:

- **`[ ]` (todo):** Reopens the issue if closed; removes the `in-progress` label if present.
- **`[/]` (in-progress):** Reopens the issue if closed; adds the `in-progress` label if missing.
- **`[x]` (done):** Closes the issue if open; removes the `in-progress` label if present.
- **Titles:** If the text of the task in `ROADMAP.md` differs from the GitHub issue title, the issue is renamed. `ROADMAP.md` is the single source of truth for titles.

Labels (`in-progress` and phase labels) are **auto-created** if they don't exist on the repository.

### CLI Arguments

| Flag | Default | Description |
|------|---------|-------------|
| `--roadmap PATH` | `./ROADMAP.md` | Path to the roadmap file |
| `--repo OWNER/NAME` | Auto-detected by `gh` | Target GitHub repository |
| `--dry-run` | Off | Preview all changes without applying them |

## Workflow Details — `roadmap-sync.yml`

```yaml
on:
  push:
    paths: ['ROADMAP.md']
    branches: [main]
  workflow_dispatch:
```

- **Triggers:** Push to `main` that modifies `ROADMAP.md`, or manual dispatch from the Actions tab.
- **Permissions:** `issues: write` and `contents: write` (to create/edit issues and push the updated roadmap).
- **Python version:** 3.12
- **Infinite-loop guard:** The job is skipped if the commit message contains `[roadmap-sync]`, preventing re-triggers from the bot's own commits.

## ROADMAP.md Format

```markdown
## Phase Name <!-- phase:label-name -->

- [ ] Task that already has an issue (#1)
- [ ] New task that needs an issue
- [/] Task currently in progress (#2)
- [x] Completed task (#3)
```

**Rules:**
- Each phase heading **must** include `<!-- phase:label-name -->` — the value becomes both the issue label and the phase identifier.
- Tasks with `(#N)` sync their state with that issue.
- Tasks **without** `(#N)` get a new issue created automatically (or linked to an existing issue with the same title).

## Setup in a New Repo

Copy these two files into your repository:

```
your-repo/
├── .github/
│   ├── scripts/
│   │   └── sync_roadmap.py    ← the sync engine
│   └── workflows/
│       └── roadmap-sync.yml   ← the GitHub Action trigger
└── ROADMAP.md                 ← your roadmap (create with the format above)
```

No configuration needed — the script auto-detects the repo from the `gh` CLI.

### Requirements

- **GitHub CLI (`gh`)** authenticated (`gh auth login`)
- **Python 3.12+** (for `int | None` union syntax and other modern features)
- The repository's `GITHUB_TOKEN` must have `issues: write` and `contents: write` permissions

## Manual Usage (Local)

```bash
# Preview what would change (no side effects)
python3 .github/scripts/sync_roadmap.py --dry-run

# Apply changes
python3 .github/scripts/sync_roadmap.py

# Use a different roadmap file
python3 .github/scripts/sync_roadmap.py --roadmap docs/ROADMAP.md

# Target a specific repo
python3 .github/scripts/sync_roadmap.py --repo owner/repo
```

## Example Workflow

1. Add a new task to `ROADMAP.md`:
   ```markdown
   - [ ] Add retry logic to download manager
   ```

2. Push to `main`.

3. The Action auto-creates issue `#46`, updates your file, and commits:
   ```markdown
   - [ ] Add retry logic to download manager (#46)
   ```

4. Later, mark it in progress:
   ```diff
   -- [ ] Add retry logic to download manager (#46)
   +- [/] Add retry logic to download manager (#46)
   ```

5. Push → issue `#46` gets the `in-progress` label.

6. When done:
   ```diff
   -- [/] Add retry logic to download manager (#46)
   +- [x] Add retry logic to download manager (#46)
   ```

7. Push → issue `#46` is closed and `in-progress` label is removed.

8. Rename the task:
   ```diff
   -- [x] Add retry logic to download manager (#46)
   +- [x] Add exponential backoff retry logic (#46)
   ```

9. Push → issue `#46` title is updated in GitHub to match the new text.
