# OpenCode Integration Guide

**Agent Implementation Guide for AI-Hub Skills**

This document provides step-by-step instructions for agents to integrate the auto-worklog skill into an OpenCode/oh-my-opencode environment.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Installation Steps](#installation-steps)
3. [AGENTS.md Configuration](#agentsmd-configuration)
4. [Environment Setup](#environment-setup)
5. [Verification](#verification)
6. [Troubleshooting](#troubleshooting)

## Prerequisites

Before starting, ensure:
- OpenCode is installed and configured
- Git is available
- User has write access to `~/.config/opencode/`

## Installation Steps

### Step 1: Clone the Repository

```bash
# Create the parent directory if it doesn't exist
mkdir -p ~/repos/personal
cd ~/repos/personal

# Clone the ai-hub repository
git clone https://github.com/GavinTomlins/ai-hub.git
```

**Expected Output:**
```
Cloning into 'ai-hub'...
remote: Enumerating objects: 45, done.
remote: Counting objects: 100% (45/45), done.
remote: Compressing objects: 100% (30/30), done.
remote: Total 45 (delta 10), reused 40 (delta 8), pack-reused 0
Receiving objects: 100% (45/45), 15.20 KiB | 2.50 MiB/s, done.
Resolving deltas: 100% (10/10), done.
```

### Step 2: Set Up Skills Directory Structure

**Architecture:** Skills are stored in a tool-agnostic location (`~/.agents/skills/`), and OpenCode symlinks to this location.

```bash
# Create the canonical skills directory (tool-agnostic location)
mkdir -p ~/.agents/skills

# Symlink skill from ai-hub repo to canonical location
ln -s ~/repos/personal/ai-hub/.agents/skills/auto-worklog \
  ~/.agents/skills/auto-worklog

# Verify the skill symlink
ls -la ~/.agents/skills/auto-worklog
# Should show: auto-worklog -> ~/repos/personal/ai-hub/.agents/skills/auto-worklog
```

### Step 2b: Point OpenCode to Canonical Skills Directory

```bash
# Check if OpenCode skills directory exists and what it is
ls -la ~/.config/opencode/skills 2>/dev/null

# CASE 1: Directory doesn't exist - create symlink
ln -s ~/.agents/skills ~/.config/opencode/skills

# CASE 2: Directory exists with existing skills - backup and symlink
mv ~/.config/opencode/skills ~/.config/opencode/skills.backup.$(date +%Y%m%d)
ln -s ~/.agents/skills ~/.config/opencode/skills
# Then move backed up skills to ~/.agents/skills/ if needed

# CASE 3: Already a symlink - verify it points correctly
readlink ~/.config/opencode/skills
# Should output: ~/.agents/skills
```

**Verify the complete structure:**
```bash
# Check canonical skills location
ls -la ~/.agents/skills/
# Should show: auto-worklog -> ~/repos/personal/ai-hub/.agents/skills/auto-worklog

# Check OpenCode skills location
ls -la ~/.config/opencode/skills
# Should show: skills -> ~/.agents/skills

# Verify skill is accessible from both locations
ls ~/.agents/skills/auto-worklog/SKILL.md
ls ~/.config/opencode/skills/auto-worklog/SKILL.md
# Both should show the file
```

**Directory Structure Overview:**
```
~/repos/personal/ai-hub/                    # Repository
└── .agents/
    └── skills/
        └── auto-worklog/                   # Skill source
            └── SKILL.md

~/.agents/skills/                           # Canonical skills (tool-agnostic)
└── auto-worklog -> ~/repos/personal/...    # Symlink to skill

~/.config/opencode/skills -> ~/.agents/skills  # OpenCode points to canonical
```

### Step 3: Create Worklog Directory

```bash
# Create the default worklog directory
mkdir -p ~/.local/share/ai-hub/worklog/

# Verify write access
touch ~/.local/share/ai-hub/worklog/.test && rm ~/.local/share/ai-hub/worklog/.test
echo "Directory is writable ✓"
```

## AGENTS.md Configuration

### Step 4: Read Current AGENTS.md

```bash
# Check if AGENTS.md exists
cat ~/.config/opencode/AGENTS.md 2>/dev/null || echo "AGENTS.md not found, will create new"
```

### Step 5: Update AGENTS.md

Add or update the following sections in `~/.config/opencode/AGENTS.md`:

```markdown
# Agent Team Configuration

## Overview

This document defines the agent team startup configuration and management policies for the OpenCode environment.

## Agent Team

### Core Agents (Always Active)

| Agent | Purpose | Model |
|-------|---------|-------|
| sisyphus | Main orchestrator | ollama/qwen2.5-coder-14b-opt:latest |
| atlas | Task execution | ollama/qwen2.5-coder-14b-opt:latest |
| explore | Codebase exploration | ollama/qwen2.5-coder-14b-opt:latest |
| hephaestus | Build specialist | ollama/qwen2.5-coder-14b-opt:latest |
| librarian | Documentation & research | ollama/qwen2.5-coder-14b-opt:latest |
| metis | Pre-planning consultant | ollama/qwen2.5-coder-14b-opt:latest |
| momus | Plan reviewer | ollama/qwen2.5-coder-14b-opt:latest |
| multimodal-looker | Image/document analysis | ollama/qwen2.5-coder-14b-opt:latest |
| oracle | Debugging consultant | ollama/qwen2.5-coder-14b-opt:latest |
| prometheus | Planning specialist | ollama/qwen2.5-coder-14b-opt:latest |

### Optional Agents (Configurable)

| Agent | Purpose | Model | Status |
|-------|---------|-------|--------|
| worklogger | Automatic worklog tracking | opencode/claude-haiku-4-5 | Always enabled |
| plannotor | Plan reviewer for submit_plan | opencode/claude-haiku-4-5 | Enabled by default |

## Startup Behavior

### Session Initialization (Every Prompt)

On every user interaction, the following agents are automatically invoked:

1. **Node Sage** (`graphiti-memory`) - Initializes memory context for the current request and makes prior context available to all agent team members.
   - Load before other specialized work to improve continuity
   - Keep executable skill identifier as `graphiti-memory`
2. **worklogger** - Initializes/updates worklog for current session
   - Detects work intent from user message
   - Creates worklog entry with timestamp and context
   - Appends to existing worklog if same day/topic

### Agent State Management

Agent state is stored in: `~/.config/opencode/config/agent-state.json`

```json
{
  "plannotator": {
    "enabled": true,
    "autoReviewOnSubmit": true
  }
}
```

## Automatic Invocation Rules

### Node Sage (`graphiti-memory`)

**Trigger**: Every user request prior to team-agent execution

**Behavior**:
1. Load memory context through `graphiti-memory`
2. Make memory context available to all team agents
3. Keep public-facing references as **Node Sage** in human-facing messaging

### worklogger

**Trigger**: Every user message that indicates work intent

**Detection patterns**:
- Explicit requests: "work on...", "implement...", "fix..."
- Review requests: "review...", "check...", "analyze..."
- Refactor requests: "refactor...", "reorganize...", "clean up..."
- Debug requests: "debug...", "investigate...", "why is..."
- Plan requests: "plan...", "design...", "architecture..."

**Behavior**:
1. Check if worklog exists for today with similar topic
2. If exists: Append new entry with timestamp
3. If not exists: Create new worklog file
4. Log context: git branch, working directory, related MRs

**Environment Configuration**:
- `AUTO_WORKLOG_DIR` - Custom worklog directory (optional)
- Default: `~/.local/share/ai-hub/worklog/`

**Worklog Path Resolution**:
1. `$AUTO_WORKLOG_DIR` (if set)
2. `~/.local/share/ai-hub/worklog/` (default)
3. `./worklogs/` (fallback)

### plannotor

**Trigger**: When `submit_plan` is called AND plannotator is enabled

**Behavior**:
1. User creates plan using standard workflow
2. Before plan is submitted to user, plannotator reviews it
3. Review results are presented alongside the plan
4. User can proceed with or without review feedback

**Configuration**:
- Default: Enabled (autoReviewOnSubmit: true)
- Can be disabled via: `/plannotator-disable`
- Can be re-enabled via: `/plannotator-enable`

## Commands

### `/plannotator-enable`
Enable plannotator integration with submit_plan.

### `/plannotator-disable`
Disable plannotator integration with submit_plan.

### `/plannotator-status`
Show current plannotator configuration status.

## Implementation Notes

### For Sisyphus (Orchestrator)

When processing user requests:

1. **Always invoke worklogger first**:
   ```typescript
   // Before any other work
   await skill("graphiti-memory") // Node Sage
   await skill("worklog-logger")
   ```

2. **All team agents should run with Node Sage memory context**:
   - Refer to the memory capability as **Node Sage** in human-facing messaging
   - Use `graphiti-memory` as the executable skill identifier

3. **Check plannotator state before submit_plan**:
   ```typescript
   if (plannotatorState.enabled && plannotatorState.autoReviewOnSubmit) {
     // Include plannotator review in submit_plan workflow
   }
   ```

4. **Respect user preferences**:
   - If user explicitly disables plannotator, don't invoke it
   - State persists across sessions

## File Locations

- Agent config: `~/.config/opencode/oh-my-opencode.json`
- Agent state: `~/.config/opencode/config/agent-state.json`
- Commands: `~/.config/opencode/command/`
- Skills (canonical): `~/.agents/skills/`
- Skills (OpenCode symlink): `~/.config/opencode/skills/ -> ~/.agents/skills/`
- This file: `~/.config/opencode/AGENTS.md`

### Directory Structure

```
~/.agents/skills/                    # Canonical skills location (tool-agnostic)
└── auto-worklog -> ~/repos/...      # Symlink to skill from ai-hub repo

~/.config/opencode/skills -> ~/.agents/skills  # OpenCode symlink to canonical

~/repos/personal/ai-hub/             # Repository
└── .agents/skills/auto-worklog/     # Skill source
    └── SKILL.md
```

## Worklog Configuration

### Auto-Worklog Skill

**Source**: `~/repos/personal/ai-hub/.agents/skills/auto-worklog/`
**Installed**: `~/.agents/skills/auto-worklog/` (symlink)
**Accessible via**: `~/.config/opencode/skills/auto-worklog/` (through symlink chain)

**Features**:
- Automatic worklog creation on work intent detection
- Git context capture (branch, commits, MRs)
- Skills, MCP servers, and models tracking
- Prompt and user feedback logging
- Timezone-aware timestamps
- Cross-platform compatibility (OpenCode, Claude, ChatGPT, n8n)

**Detection Logic**:
The skill detects work intent from user messages containing keywords like:
- "work on...", "implement...", "fix..."
- "review...", "check...", "analyze..."
- "refactor...", "reorganize...", "clean up..."
- "debug...", "investigate...", "why is..."
- "plan...", "design...", "architecture..."

**Worklog Format**:
```
~/.local/share/ai-hub/worklog/YYYY-MM-DD {Topic}.md
```

## Maintenance

- Update this document when adding new agents
- Modify agent-state.json to change default behaviors
- Agent configurations are loaded on OpenCode startup
- Worklog skills are sourced from ai-hub repository via `~/.agents/skills/` (canonical location)
- OpenCode skills directory is a symlink to the canonical location
- To add new skills, symlink from repository to `~/.agents/skills/{skill-name}/`

---

*Generated by ai-hub integration guide*
*Repository: https://github.com/GavinTomlins/ai-hub*
```

### Step 6: Apply Changes

Save the AGENTS.md file. OpenCode will pick up changes on the next interaction.

## Environment Setup

### Step 7: Set Environment Variable (Optional but Recommended)

Add to shell configuration for persistence:

```bash
# For zsh
echo 'export AUTO_WORKLOG_DIR="$HOME/.local/share/ai-hub/worklog/"' >> ~/.zshrc
source ~/.zshrc

# For bash
echo 'export AUTO_WORKLOG_DIR="$HOME/.local/share/ai-hub/worklog/"' >> ~/.bashrc
source ~/.bashrc
```

### Step 8: Verify Environment

```bash
# Check environment variable
echo $AUTO_WORKLOG_DIR
# Expected: /Users/<username>/.local/share/ai-hub/worklog/

# Check directory exists
ls -la ~/.local/share/ai-hub/worklog/
# Expected: Directory listing (empty is OK)
```

## Verification

### Step 9: Test Worklog Creation

Start a new OpenCode session and trigger work intent:

```
User: Work on implementing a new feature
```

**Expected Agent Behavior:**
1. Agent should respond with: "📝 Worklog initialized: ~/.local/share/ai-hub/worklog/YYYY-MM-DD Implementing a new feature.md"
2. A new markdown file should be created
3. File should contain session context, timestamp, and user request

### Step 10: Verify File Creation

```bash
# List recent worklogs
ls -lt ~/.local/share/ai-hub/worklog/ | head -5

# View the latest worklog
cat ~/.local/share/ai-hub/worklog/$(ls -t ~/.local/share/ai-hub/worklog/ | head -1)
```

**Expected Output Structure:**
```markdown
# Worklog: Implementing a new feature

**Session ID:** ses_xxx
**Date:** YYYY-MM-DD
**Time:** HH:MM:SS
**Timezone:** Australia/Brisbane
**Agent:** Sisyphus
**Status:** 🟡 In Progress

## Prompt and User Feedback

### Initial Prompt
```
Work on implementing a new feature
```

## Request Context
...
```

## Troubleshooting

### Issue: Skill not found

**Symptom:** Agent doesn't recognize the worklog skill

**Solution:**
```bash
# Check both symlink locations
ls -la ~/.agents/skills/auto-worklog
ls -la ~/.config/opencode/skills/auto-worklog

# If canonical location missing, recreate:
ln -s ~/repos/personal/ai-hub/.agents/skills/auto-worklog \
  ~/.agents/skills/auto-worklog

# If OpenCode location is wrong, fix it:
rm ~/.config/opencode/skills 2>/dev/null
ln -s ~/.agents/skills ~/.config/opencode/skills
```

### Issue: OpenCode skills directory is a regular directory instead of symlink

**Symptom:** Skills installed but OpenCode doesn't see them

**Solution:**
```bash
# Backup existing skills
mv ~/.config/opencode/skills ~/.config/opencode/skills.backup.$(date +%Y%m%d)

# Create symlink to canonical location
ln -s ~/.agents/skills ~/.config/opencode/skills

# If you had skills in the backup, move them to canonical location
mv ~/.config/opencode/skills.backup.*/auto-worklog ~/.agents/skills/ 2>/dev/null
```

### Issue: Broken symlinks

**Symptom:** "No such file or directory" when accessing skills

**Solution:**
```bash
# Find and remove broken symlinks
find ~/.agents/skills -type l ! -exec test -e {} \; -print
find ~/.agents/skills -type l ! -exec test -e {} \; -delete

# Recreate symlinks
ln -s ~/repos/personal/ai-hub/.agents/skills/auto-worklog \
  ~/.agents/skills/auto-worklog

# Verify repository exists
ls ~/repos/personal/ai-hub/.agents/skills/auto-worklog/SKILL.md
```

### Issue: Permission denied

**Symptom:** Cannot write to worklog directory

**Solution:**
```bash
# Check directory permissions
ls -ld ~/.local/share/ai-hub/worklog/

# Fix permissions
chmod 755 ~/.local/share/ai-hub/worklog/

# Verify ownership
ls -la ~/.local/share/ai-hub/ | grep worklog
```

### Issue: Worklog not auto-triggering

**Symptom:** No worklog created on work intent

**Solution:**
1. Verify AGENTS.md has worklogger in Startup Behavior
2. Check that detection patterns match user message
3. Ensure skill file is readable:
   ```bash
   cat ~/.config/opencode/skills/auto-worklog/SKILL.md | head -20
   ```

### Issue: Wrong worklog directory

**Symptom:** Worklogs created in unexpected location

**Solution:**
```bash
# Check environment variable
echo $AUTO_WORKLOG_DIR

# Set correctly if needed
export AUTO_WORKLOG_DIR="$HOME/.local/share/ai-hub/worklog/"

# Or update shell profile
echo 'export AUTO_WORKLOG_DIR="$HOME/.local/share/ai-hub/worklog/"' >> ~/.zshrc
```

## Next Steps

1. **Test with various work patterns** - Try different detection keywords
2. **Customize worklog template** - Edit SKILL.md to add/remove fields
3. **Set up backup** - Consider backing up worklog directory
4. **Explore other skills** - Check ai-hub repository for additional skills

## Agent Checklist

When implementing this integration, ensure:

- [ ] Repository cloned to `~/repos/personal/ai-hub/`
- [ ] Canonical skills directory created at `~/.agents/skills/`
- [ ] Skill symlinked to `~/.agents/skills/auto-worklog/`
- [ ] OpenCode skills directory (`~/.config/opencode/skills`) points to `~/.agents/skills`
- [ ] Symlinks verified working from both locations
- [ ] Worklog directory created at `~/.local/share/ai-hub/worklog/`
- [ ] AGENTS.md updated with worklogger configuration
- [ ] Environment variable set (optional but recommended)
- [ ] Test worklog creation successful
- [ ] File permissions correct
- [ ] Documentation updated if needed

---

**Integration Guide Version:** 1.0  
**Last Updated:** 2025-03-15  
**Repository:** https://github.com/GavinTomlins/ai-hub
