---
name: backup
description: Helps create and manage backups so you can safely return to any previous point in a project. Use this skill when the user mentions backup, restore, checkpoint, rollback, undo, safety copy, versioning, or "I want to be able to go back to". Also use when starting a risky operation like major refactoring, bulk changes, deleting files, or restructuring a project. The skill works for any file-based project — code, LaTeX, documents, data pipelines, or research.
---

# Backup — Safe Checkpoints for Any Project

A skill for creating reliable backups so you can always return to a known good state.
Principle: **A backup is only useful if you can find it, understand it, and restore from it.**

---

## The Core Problem

Most projects fail at backup in one of three ways:
1. **No backup** — "I'll do it later"
2. **Backup with bad names** — `backup_final_v3_REAL_2.zip` tells you nothing
3. **Backup in the wrong place** — backed up to the same drive that fails

---

## When to Take a Backup

Take a backup **before** any of these:
- A major structural change (moving files, renaming, reorganising)
- Bulk automated operations (scripts that modify many files at once)
- Deleting anything you might want later
- A new session after a long pause (captures current known-good state)
- Before upgrading dependencies, tools, or formats
- When something is working and you are about to change it

Rule of thumb: **if you would be upset to lose the current state, back it up first.**

---

## Naming Convention

Always include three elements: **date + version/label + what state**

```
BACKUP_YYYYMMDD_v{N}_{label}/
```

Examples:
```
BACKUP_20260316_v25_before-print-fix/
BACKUP_20260315_v19_after-chapter-numbering/
BACKUP_20260314_komplet-working/
```

Never use:
- `backup_final` — final is never final
- `backup_new` — new becomes old immediately
- `backup2` — tells you nothing about content or timing

---

## What to Include in a Backup

**Always include:**
- All source files that are actively edited
- Configuration files (build scripts, settings)
- The current output/result (PDF, compiled binary, etc.)
- Any log or status files

**Do NOT need to include:**
- Auto-generated files that can be regenerated (compiled objects, node_modules, LaTeX aux files)
- Files that are already safely stored elsewhere (e.g. already pushed to git)

**For LaTeX projects specifically:**
- Include: `*_body.tex`, `main.tex`, `references.bib`, `*.json` data files
- Exclude: `*.aux`, `*.log`, `*.toc`, `*.out`, `*.synctex.gz`

---

## Where to Store Backups

Use **two locations** — at least one off the primary work machine:

| Location | Purpose | Risk |
|----------|---------|------|
| Same machine, different folder | Fast restore | Lost if machine fails |
| Cloud (iCloud, Dropbox, Drive) | Off-machine safety | Requires sync |
| Git remote (GitHub) | Version history + off-machine | Requires commit discipline |
| External drive | Physical separation | Easy to forget |

**Minimum viable:** same machine folder + cloud sync running automatically.

---

## Restore Procedure

When you need to go back:

1. **Find the right backup** — use the name to identify the correct point
2. **Do not overwrite** — copy backup to a new location, not back onto active files
3. **Verify before switching** — open/compile/run the backup to confirm it works
4. **Keep the broken state** — rename current files to `BROKEN_YYYYMMDD/` before restoring
5. **Document why** — add one line to your log: "Restored from BACKUP_X because Y"

---

## Quick Backup Script (Mac/Linux)

```bash
# One-line backup
cp -r ./project "BACKUP_$(date +%Y%m%d)_LABEL"

# For LaTeX: backup only source files, not generated files
rsync -av --exclude='*.aux' --exclude='*.log' --exclude='*.toc' \
  ./project/ "BACKUP_$(date +%Y%m%d)_v25_before-changes/"
```

---

## Verification Checklist

After taking a backup, verify:
- [ ] Can you navigate to the backup folder and find the files?
- [ ] Does the backup open/compile/run correctly?
- [ ] Is the name meaningful enough to find in 3 months?
- [ ] Is it stored in at least two locations?

---

## Common Mistakes

| Mistake | Consequence | Fix |
|---------|-------------|-----|
| Backing up to same folder as work | Overwritten if project folder corrupted | Use separate folder or drive |
| Not testing the backup | Discover it is broken when you need it | Always verify after creating |
| Too many backups, no naming system | Cannot find the right point | Prune and name consistently |
| Backing up only some files | Restore is incomplete | Use a checklist of what to include |
| Cloud sync instead of backup | Deletions sync too | Use versioned copies, not just sync |

---

**Single most important rule:** Take the backup *before* the change, not after.
