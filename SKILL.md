---
name: mac-disk-clean
description: >-
  Systematic, safe Mac disk cleanup and telemetry runbook. Uses Mole (mo) by tw93 (https://github.com/tw93/mole)
  to audit and clean user app caches, browser models, package manager stores, and project build artifacts.
  Includes safety whitelists for personal data, deep analysis of agent worktrees (~/.claude/jobs),
  and auto-generates rich HTML dashboards for visualization.
---

# 🚀 Mac Disk Cleanup & Optimization Runbook
> **Instructions for the AI Assistant / LLM reading this file:**
> You are acting as an autonomous systems engineer. The user has provided this runbook to perform a safe, structured disk cleanup and optimization on this Mac.
> Execute each phase in order, strictly enforce all data safety whitelists, and present the final results as an interactive HTML dashboard opened in the user's default browser.

---

## 🔒 Cardinal Safety Rules (CRITICAL — NEVER VIOLATE)

Before running any deletions, confirm and protect the following directories:

1. **iMessage History & Attachments:**
   - 🛑 **NEVER** delete or modify `~/Library/Messages/chat.db` or `~/Library/Messages/Attachments`.
   - ✅ **Safe:** `~/Library/Messages/StickerCache/*` and `~/Library/Messages/Caches/Previews/*` (auto-generated preview thumbnails only).

2. **iCloud Drive & Cloud Sync:**
   - 🛑 **NEVER** touch `~/Library/Mobile Documents/` or `~/Library/CloudStorage/`.
   - All cloud-synced storage trees are strictly whitelisted and protected.

3. **Photos Library:**
   - 🛑 **NEVER** delete or modify `~/Pictures/` or `*.photoslibrary` packages.
   - ✅ **Safe:** `Photos.cache` inside Address Book contact card icons.

4. **Agentic Memory & Configurations:**
   - 🛑 **NEVER** delete `~/.claude/projects/`, `~/.claude/skills/`, `~/.claude/plugins/`, `~/.gemini/config/`, or conversation histories.
   - ✅ **Safe:** Completed/stale subagent worktrees inside `~/.claude/jobs/*/tmp`.

---

## 🛠️ Step 0: Tooling & Telemetry Baseline

Mole (`mo`) is the official native Mac cleaner tool created by **tw93** ([GitHub: tw93/mole](https://github.com/tw93/mole) · [mole.fit](https://mole.fit)):

```bash
# 1. Ensure latest Mole is installed / updated via Homebrew
if ! command -v mo &>/dev/null; then
  echo "Installing latest Mole from Homebrew..."
  brew install mole || brew install tw93/mole/mole
else
  echo "Checking for latest Mole updates..."
  brew upgrade mole 2>/dev/null || brew upgrade tw93/mole/mole 2>/dev/null || true
fi

# 2. Check initial system telemetry and record starting free space
mo status
df -h / | tail -1
```

---

## ⚡ Phase 1: Zero-Risk System & Application Caches

Cleans temporary user application caches, browser on-device AI models, and thumbnail caches:

```bash
# 1. Preview safe cache cleanup
mo clean --dry-run

# 2. Execute user-level cleanup (automatically skips sudo tasks if unprivileged)
mo clean
```

*Expected recovery: 15–40 GB of auto-regenerating cache data.*

---

## 📦 Phase 2: Project Build Artifact Purge

Scans active and inactive development repositories (`~/Code`, `~/dev`, `~/.claude/worktrees`, etc.) for stale build artifacts:

```bash
# 1. Preview stale build directories
mo purge --dry-run

# 2. Purge stale node_modules, .next, .venv, dist, and coverage folders
mo purge
```

*Expected recovery: 10–30 GB across idle repositories (re-generated on demand via `npm install` / `pip install`).*

---

## 🐳 Phase 3: Package Managers & Virtual Containers

Cleans global package store caches and container disk images:

```bash
# 1. Prune unreferenced pnpm packages
pnpm store prune 2>/dev/null || true

# 2. Clean npm cache
npm cache clean --force 2>/dev/null || true

# 3. Clean bun and uv caches
bun pm cache rm 2>/dev/null || true
uv cache clean 2>/dev/null || true

# 4. Prune Docker Desktop virtual machine layers (if Docker is running)
docker system prune -a --volumes -f 2>/dev/null || true

# 5. Clean stale installer DMGs / PKGs in Homebrew cache
mo installer 2>/dev/null || true
```

---

## 🔬 Phase 4: Deep Directory Breakdown & Agent Worktrees

Pinpoint large directories across the user's home folder:

```bash
# 1. Inspect top directory distribution in ~
du -d 1 -h ~ 2>/dev/null | sort -hr | head -15

# 2. Inspect Claude Code job accumulation
du -sh ~/.claude/jobs 2>/dev/null || true
```

### Safe Agent Cleanup Rule:
- If `~/.claude/jobs/` exceeds 20 GB, delete completed job worktrees older than 14 days while keeping memories and project states intact:
  ```bash
  find ~/.claude/jobs -maxdepth 1 -mindepth 1 -type d -mtime +14 -exec rm -rf {} +
  ```

---

## 📊 Phase 5: Interactive HTML Dashboard Deliverable

After completing the cleanup, measure the final disk state and compile a self-contained, styled HTML report:

```bash
# Measure final disk free space
df -h / | tail -1
mo status
```

### Dashboard Requirements:
1. **Design**: Dark theme with glassmorphism cards, glowing status badges, and Google Fonts (`Outfit`, `Inter`).
2. **Metrics Grid**: Display **Starting Free Space**, **Final Free Space**, **Total Reclaimed Space**, and **Health Score**.
3. **Itemized Table**: Breakdown of space saved across categories (Project artifacts, App caches, Browsers, Package stores).
4. **Safety Confirmation**: List verified protected personal stores (iMessage `chat.db`, iCloud Drive, Photos Library).
5. **Auto-Launch**: Save the report as `report.html` and automatically open it in the default browser:
   ```bash
   open report.html
   ```
