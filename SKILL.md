---
name: mac-disk-clean
description: >-
  Safe Mac disk cleanup runbook. Uses Mole (mo) by tw93 (https://github.com/tw93/mole)
  to audit and clean user app caches, browser models, package manager stores, and
  project build artifacts. Includes protection rules for personal data, analysis of
  agent worktrees (~/.claude/jobs), and a summary dashboard.
---

# Mac Disk Cleanup Runbook

## Instructions for the AI assistant

You are a systems engineer helping the user clean their Mac. Follow these rules strictly.

**Preview-first contract.** The default mode is PREVIEW ONLY. For every phase:

1. Run the dry-run or listing command first.
2. Show the user the list of items and the estimated size.
3. Wait for the user to say "yes" in the chat before running the deleting command.
4. Never run a preview and a delete in the same command block.
5. Never use `-f`, `--force`, or `-y` to skip a confirmation prompt.
6. If the user says "just do it all", still confirm once per phase. Show the preview, state what will be deleted, and wait for "yes".
7. Recovery numbers in this runbook are typical, not promised. Caches rebuild on demand, but rebuilding costs time and bandwidth.

If the user declines a phase, skip it and move on.

---

## Protected data

Before running any deletion, confirm that the following paths are never modified or removed.

| Protected data | Rule |
|---|---|
| iMessage history and attachments | Never delete or modify `~/Library/Messages/chat.db` or `~/Library/Messages/Attachments`. Safe to clear: `~/Library/Messages/StickerCache/*` and `~/Library/Messages/Caches/Previews/*` (auto-generated thumbnails only). |
| iCloud Drive and cloud sync | Never touch `~/Library/Mobile Documents/` or `~/Library/CloudStorage/`. |
| Photos library | Never delete or modify `~/Pictures/` or `*.photoslibrary` packages. |
| Agent memories and configurations | Never delete `~/.claude/projects/`, `~/.claude/skills/`, `~/.claude/plugins/`, `~/.gemini/config/`, or conversation histories. Safe to clean: completed stale worktrees inside `~/.claude/jobs/*/tmp` (with user approval). |

---

## Step 0: Tooling and telemetry baseline

Mole (`mo`) is a native Mac cleaner created by tw93 (https://github.com/tw93/mole, https://mole.fit). It is distributed under GPL-3.0. This runbook orchestrates Mole and adds guard rails. It is not affiliated with or endorsed by tw93.

Do not install or upgrade anything without approval.

```bash
# 1. Check whether Mole is installed. Do not install or upgrade yet.
if command -v mo >/dev/null 2>&1; then
  mo --version
else
  echo "Mole is not installed. Install command: brew install mole"
fi
```

If Mole is missing, show the user the install command and wait for "yes" before running `brew install mole`. If Mole is present, do not upgrade it.

```bash
# 2. Record starting disk state
mo status
df -h / | tail -1
```

Record the starting free space value. You will compare it to the final value in Phase 5.

---

## Phase 1: System and application caches

Cleans temporary user application caches, browser on-device AI models, and thumbnail caches.

**Step 1: Preview.** Run the dry-run and show the user the list.

```bash
mo clean --dry-run
```

**Step 2: Wait for approval.** Show the user what will be removed and the estimated size. Do not proceed until the user says "yes".

**Step 3: Execute.** Only after the user approves:

```bash
mo clean
```

Typical recovery: 15 to 40 GB of auto-regenerating cache data. Actual results vary.

---

## Phase 2: Project build artifact purge

Scans development repositories (`~/Code`, `~/dev`, `~/.claude/worktrees`, etc.) for stale build artifacts.

**Step 1: Preview.** Run the dry-run and show the user the list.

```bash
mo purge --dry-run
```

**Step 2: Wait for approval.** Show the user the directories and sizes. Do not proceed until the user says "yes".

**Step 3: Execute.** Only after the user approves:

```bash
mo purge
```

Typical recovery: 10 to 30 GB across idle repositories. These are re-generated on demand via `npm install`, `pip install`, etc. Actual results vary.

---

## Phase 3: Package managers and containers

Cleans global package store caches and container layers.

**Step 1: Preview package manager caches.** Show the user the estimated sizes before cleaning.

```bash
# Check pnpm store size
du -sh "$(pnpm store path 2>/dev/null)" 2>/dev/null || true

# Check npm cache size
du -sh ~/.npm 2>/dev/null || true

# Check bun cache
du -sh ~/.bun/install/cache 2>/dev/null || true

# Check uv cache
uv cache dir 2>/dev/null && du -sh "$(uv cache dir)" 2>/dev/null || true
```

**Step 2: Wait for approval.** Show the user the sizes and explain these are re-downloaded on demand. Do not proceed until the user says "yes".

**Step 3: Execute package manager cleanup.** Only after the user approves:

```bash
pnpm store prune 2>/dev/null || true
# npm requires --force here. It is not a prompt skip. Run it only after the user said yes.
npm cache clean --force 2>/dev/null || true
bun pm cache rm 2>/dev/null || true
uv cache clean 2>/dev/null || true
```

**Step 4: Docker (if running).** Handle Docker separately. First, show the user the current Docker disk usage.

```bash
docker system df 2>/dev/null || true
docker image ls 2>/dev/null || true
docker volume ls 2>/dev/null || true
```

Explain to the user: `docker system prune` removes stopped containers, unused networks, dangling images, and build cache. It does NOT remove named volumes. The `--volumes` flag would also remove named volumes, which may contain databases and persistent application data. This runbook does not use `--volumes`. Ask before adding `-a`. With `-a` Docker also removes every unused tagged image, such as node, python, or postgres base images. They must be downloaded again later.

**Step 5: Wait for Docker approval.** Do not proceed until the user says "yes".

**Step 6: Execute Docker cleanup.** Only after the user approves:

```bash
docker system prune
```

Do not add `--volumes` or `-f`. If the user wants to remove a specific volume, they must name it explicitly. Remove only the volumes the user names, one at a time:

```bash
docker volume rm <volume-name>
```

**Step 7: Preview the Homebrew download cache.** Show the user the size.

```bash
du -sh "$(brew --cache)" 2>/dev/null || true
```

**Step 8: Wait for the user to say "yes".**

**Step 9: Clean the installer cache.** Only after approval.

```bash
mo installer
```

---

## Phase 4: Deep directory breakdown and agent worktrees

Identify large directories across the home folder.

**Step 1: Survey.**

```bash
du -d 1 -h ~ 2>/dev/null | sort -hr | head -15
du -sh ~/.claude/jobs 2>/dev/null || true
```

**Step 2: Agent worktree audit.** If `~/.claude/jobs/` is large, list candidate directories older than 14 days.

```bash
find ~/.claude/jobs -maxdepth 1 -mindepth 1 -type d -mtime +14 -print 2>/dev/null
```

Then show the size of each candidate:

```bash
# For each directory found above, run:
du -sh <directory>
```

Present the list to the user. Explain the following:

- Directory mtime reflects when the directory itself was last modified, not when files inside it were last accessed. A directory may appear stale by mtime but still contain recent work.
- Job directories may hold unpushed code, uncommitted changes, or work in progress.
- The user should review the list and name which directories to remove.

**Step 3: Wait for the user to name specific directories.** Do not remove any directory the user has not named.

**Step 4: Remove only the directories the user named.** Run one command per directory. Do not use `-f`. Check for unpushed work first: run `git -C "<directory>" status --short` on any repository inside it, and show the result.

```bash
rm -r "<directory-the-user-named>"
```

---

## Phase 5: Summary dashboard

After completing the cleanup, measure the final disk state.

```bash
df -h / | tail -1
mo status
```

Compile the results into a self-contained HTML report. The dashboard is a summary the assistant writes from the command outputs. The user should trust the terminal output over the dashboard if there is any discrepancy.

### Dashboard contents:

1. **Design:** Dark theme, clean layout. Use Google Fonts (Outfit, Inter) if desired.
2. **Metrics grid:** Starting free space, final free space, total reclaimed space, health score.
3. **Itemized table:** Breakdown of space saved across categories (project artifacts, app caches, browsers, package stores).
4. **Safety confirmation:** List the verified protected personal stores (iMessage chat.db, iCloud Drive, Photos library).
5. **Auto-launch:** Save the report as `report.html` and open it:

```bash
open report.html
```
