# mac-disk-clean

**The LLM-guided Mac disk cleaner. 50+ GB reclaimed safely, zero subscription required.**

An autonomous, agentic runbook for developers who want to clean, audit, and optimize their Mac without paying for opaque commercial subscription cleaner apps. Powered by [tw93/mole](https://github.com/tw93/mole), with hardcoded safety whitelists for personal data and automated before/after HTML dashboard generation.

---

## Why this exists

Commercial Mac cleaners charge $40+/year subscriptions for basic cache purges while missing the modern cruft that actually eats developer storage:

- **Subagent Worktree Bloat:** Hundreds of gigabytes archived in `~/.claude/jobs/` or `~/.codex/`
- **Stale Monorepo Build Trees:** Inactive `node_modules`, `.next`, `.venv`, and coverage folders across git worktrees
- **Package Manager Stores:** Dangling virtual packages in `pnpm`, `bun`, `uv`, and `npm`
- **Browser AI Models:** Hidden on-device LLM classifier weights in Chrome and Brave

**mac-disk-clean** gives your AI assistant (Claude Code, Gemini, ChatGPT, Codex) a safe, systematic runbook to inspect, preview, and clean your Mac while **strictly protecting your personal data**.

---

## 🔒 Cardinal Safety Guarantees

The runbook enforces strict protection gates before any deletion:

| Protected Data | Safety Policy |
|---|---|
| **iMessage History & Attachments** | 🛑 **NEVER** touched (`chat.db` & `~/Library/Messages/Attachments` preserved; only generated link thumbnails cleared) |
| **iCloud Drive & Cloud Sync** | 🛑 **NEVER** touched (`~/Library/Mobile Documents` & `CloudStorage` whitelisted) |
| **Photos Library** | 🛑 **NEVER** touched (`~/Pictures` & `*.photoslibrary` preserved) |
| **Agent Memories & Skills** | 🛑 **NEVER** touched (`~/.claude/projects`, `~/.claude/skills`, `~/.gemini/config` preserved) |

---

## 🚀 Quick Start (Drag & Drop)

### Method 1: Drop into any AI Chat
Drag [`SKILL.md`](./SKILL.md) directly into **Claude Code, ChatGPT, Gemini, or Codex**, and tell it:
```text
Please execute this Mac Disk Cleanup runbook on my machine.
```

### Method 2: Install as a Claude Code Skill
```bash
# Add directly to your Claude skills
mkdir -p ~/.claude/skills/mac-disk-clean
curl -sSL https://raw.githubusercontent.com/dvaladares/mac-disk-clean/main/SKILL.md > ~/.claude/skills/mac-disk-clean/SKILL.md
```

### Method 3: Antigravity / agy CLI
```bash
mkdir -p ~/.gemini/config/skills/mac-disk-clean
curl -sSL https://raw.githubusercontent.com/dvaladares/mac-disk-clean/main/SKILL.md > ~/.gemini/config/skills/mac-disk-clean/SKILL.md
```

---

## 📋 What the Runbook Executes

```
1. Tooling Baseline     → Verifies / installs `tw93/mole` via Homebrew & records telemetry
2. Phase 1: Caches      → User app caches, browser AI models, service workers (`mo clean`)
3. Phase 2: Build Trees → Inactive node_modules, .next, .venv in git repos (`mo purge`)
4. Phase 3: Packages    → pnpm virtual store, npm, bun, uv, and Docker VM layers
5. Phase 4: Agent Audit → Scans ~/.claude/jobs/ and deep home directory storage
6. Phase 5: Dashboard   → Automatically renders & opens an interactive HTML telemetry report
```

---

## 📊 Sample Output Dashboard

Every cleanup compiles a self-contained, dark-mode **HTML Telemetry Dashboard** and opens it in your default browser:

- **Visual APFS Gauge:** Live before-and-after storage allocation
- **Health Score & Metrics:** Reclaimed space, disk headroom, RAM pressure
- **Itemized Space Breakdown:** Categorized list of all pruned artifacts

---

## 🙏 Credits & Upstream Engine

This runbook orchestrates and builds upon the fast, native macOS utility [Mole](https://github.com/tw93/mole) created by [Tw93](https://github.com/tw93) ([mole.fit](https://mole.fit)). Mole is distributed under the GNU General Public License v3.0 (GPL-3.0).

---

## ⚖️ License & Disclaimer

- **License:** [MIT License](./LICENSE) · Copyright (c) 2026 Daniel Valadares
- **Disclaimer:** All product names, logos, and brands are property of their respective owners. CleanMyMac is a trademark of MacPaw Inc. Mention of third-party products is solely for descriptive and comparative purposes.
