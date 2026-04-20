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
- **Before any agentic AI session (Cowork etc.) — always, without exception**

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
- **Working data files: refs_honest.json, *.xlsx audit files, PROJECT_LOG.md**

**Do NOT need to include:**
- Auto-generated files that can be regenerated (compiled objects, `node_modules`, LaTeX aux files)
- Files that are already safely stored elsewhere (e.g. already pushed to git)

**For LaTeX projects specifically:**
- Include: `*_standalone.tex`, `main.tex`, `references.bib`, `*.json` data files, `PROJECT_LOG.md`
- Exclude: `*.aux`, `*.log`, `*.toc`, `*.out`, `*.synctex.gz`, `*.bbl`, `*.bcf`, `*.blg`

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

## ⚠️ Backup Scripts — Path Safety

**The bash scripts below use relative paths (`./project`) which are safe for
local folders. If your source files are in a cloud path with spaces (e.g. iCloud's
`/Users/.../Library/Mobile Documents/...`), do NOT use bash directly.**

### Safe for local paths (no spaces):
```bash
# One-line backup
cp -r ./project "BACKUP_$(date +%Y%m%d)_LABEL"

# For LaTeX: source files only, no generated files
rsync -av \
  --exclude='*.aux' --exclude='*.log' --exclude='*.toc' \
  --exclude='*.out' --exclude='*.bbl' --exclude='*.bcf' \
  --exclude='*.blg' --exclude='*.synctex.gz' \
  ./project/ "BACKUP_$(date +%Y%m%d)_v25_before-changes/"
```

### Required for iCloud paths or any path with spaces:
```python
import shutil, os
from datetime import datetime

src = "/Users/petermalmkjaer/Library/Mobile Documents/com~apple~CloudDocs/Claude/PM_Textbook"
label = "before-changes"
dst = f"/Users/petermalmkjaer/Desktop/BACKUP_{datetime.now().strftime('%Y%m%d')}_{label}"

# Extensions to exclude (LaTeX generated files)
EXCLUDE = {'.aux', '.log', '.toc', '.out', '.bbl', '.bcf', '.blg', '.synctex.gz'}

os.makedirs(dst, exist_ok=True)
for root, dirs, files in os.walk(src):
    # Skip backup subfolders to avoid recursive backup
    dirs[:] = [d for d in dirs if not d.startswith('BACKUP_')]
    for fname in files:
        if os.path.splitext(fname)[1] not in EXCLUDE:
            rel = os.path.relpath(os.path.join(root, fname), src)
            dest_file = os.path.join(dst, rel)
            os.makedirs(os.path.dirname(dest_file), exist_ok=True)
            shutil.copy2(os.path.join(root, fname), dest_file)

print(f"✓ Backup created: {dst}")
print(f"  Files: {sum(len(f) for _,_,f in os.walk(dst))}")
```

**How to detect if your path has spaces:**
```python
path = "/your/project/path"
if " " in path:
    print("⚠️  Use Python backup script — bash will fail on this path.")
else:
    print("✓ Bash backup is safe.")
```

---

## ⚠️ Upload Warning — Identical Filenames

If you upload two files with the same filename in the same session (e.g. a backup
copy and the current version of `kap01_standalone.tex`), the upload system silently
overwrites the earlier file. You will lose one of them with no warning.

**Prevention:** Rename before uploading:
- `kap01_standalone_backup.tex`
- `kap01_standalone_current.tex`

**Detection if already uploaded:** Compare file sizes — if backup and current are
identical in size, the overwrite has occurred. Restore from your local backup folder.

---

## Verification Checklist

After taking a backup, verify ALL of these before continuing:
- [ ] Navigate to the backup folder — are the files actually there?
- [ ] For LaTeX: run a **full four-pass build** from the backup copy
  (`xelatex → biber → xelatex → xelatex`) — one pass is not sufficient to verify
  that bibliography and cross-references are intact
- [ ] For data files (JSON, Excel): open and confirm record count matches expectation
- [ ] Is the name meaningful enough to find in 3 months?
- [ ] Is it stored in at least two locations?

**Why four passes matter:** A single `xelatex` run can complete without errors even
if the bibliography is broken. Only a full four-pass build confirms that biber,
cross-references, and the table of contents are all correct.

---

## Restore Procedure

When you need to go back:

1. **Find the right backup** — use the name to identify the correct point
2. **Do not overwrite** — copy backup to a new location, not back onto active files
3. **Verify before switching** — run full four-pass build from the backup
4. **Keep the broken state** — rename current files to `BROKEN_YYYYMMDD/` before restoring
5. **Document why** — add one line to your log: "Restored from BACKUP_X because Y"

---

## Common Mistakes

| Mistake | Consequence | Fix |
|---------|-------------|-----|
| Backing up to same folder as work | Overwritten if project folder corrupted | Use separate folder or drive |
| Not testing the backup | Discover it's broken when you need it | Always run full four-pass build to verify |
| Too many backups, no naming system | Can't find the right point | Prune + name consistently |
| Backing up only some files | Restore is incomplete | Use checklist — include JSON/Excel data files |
| Cloud "sync" instead of backup | Deletions sync too | Use versioned copies, not just sync |
| Using bash on iCloud paths with spaces | Silent copy failure | Use Python shutil instead |
| Uploading same filename twice | Earlier file silently overwritten | Rename files before uploading |
| Skipping backup before agentic session | Cowork/agent errors unrecoverable | Always backup before agentic work |

---

**Single most important rule:** Take the backup *before* the change, not after.
