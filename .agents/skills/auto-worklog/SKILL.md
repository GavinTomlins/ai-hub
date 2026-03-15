---
name: auto-worklog
description: AUTOMATIC worklog initialization - creates worklog entry at start of EVERY session. Use this as a meta-skill to ensure all work is logged. Add to AGENTS.md for automatic invocation.
author: Gavin Tomlins
email: gavintomlins+ai-hub@gmail.com
repository: https://github.com/GavinTomlins/ai-hub
license: Apache-2.0
compatibility:
  platforms:
    opencode:
      status: full
      auto_trigger: true
      notes: Native AGENTS.md integration
    claude_code:
      status: partial
      auto_trigger: false
      notes: Requires manual skill() call, AGENTS.md not supported
    claude_desktop:
      status: partial
      auto_trigger: false
      notes: Requires MCP server setup, limited context access
    chatgpt:
      status: partial
      auto_trigger: false
      notes: Via custom GPT with Actions, requires API integration
    n8n:
      status: partial
      auto_trigger: false
      notes: Via n8n workflow node, file system access required
    openclaw:
      status: experimental
      auto_trigger: false
      notes: Pending OpenClaw skill system implementation
  min_opencode_version: "0.5.0"
metadata:
  audience: developers
  workflow: auto-logging
  auto-trigger: true
---

# Automatic Worklog Initialization

**Author:** Gavin Tomlins (gavintomlins+ai-hub@gmail.com)  
**Repository:** https://github.com/GavinTomlins/ai-hub  
**License:** Apache 2.0

This skill automatically creates worklog entries at the start of every session.

## Cross-Platform Compatibility

This skill is designed to work across multiple AI platforms with varying levels of integration:

| Platform | Status | Auto-Trigger | Setup Required |
|----------|--------|--------------|----------------|
| **OpenCode** | ✅ Full | Yes | AGENTS.md |
| **Claude Code** | ⚠️ Partial | No | Manual skill() call |
| **Claude Desktop** | ⚠️ Partial | No | MCP server |
| **ChatGPT** | ⚠️ Partial | No | Custom GPT + API |
| **n8n** | ⚠️ Partial | No | Workflow node |
| **OpenClaw** | 🧪 Experimental | No | Pending skill system |

### Platform-Specific Setup

#### OpenCode (Primary)
```markdown
# Add to AGENTS.md
## Startup Behavior
- worklogger: Automatic worklog tracking
```
The skill auto-triggers on session start when work intent is detected.

#### Claude Code
```bash
# Manual invocation required
claude > /skill auto-worklog
claude > Topic: Implementing feature X
```
Limitations: No AGENTS.md support, requires explicit skill call

#### Claude Desktop
Requires MCP server setup:
1. Install worklog MCP server from repo
2. Configure in Claude Desktop settings
3. Use natural language: "Start worklog for task X"

#### ChatGPT
Requires Custom GPT with Actions:
1. Import skill definition as GPT instructions
2. Configure file system API endpoint
3. Use: "Initialize worklog for [topic]"

#### n8n
Workflow node configuration:
```json
{
  "skill": "auto-worklog",
  "parameters": {
    "topic": "{{ $json.topic }}",
    "worklogDir": "{{ $env.WORKLOG_DIR }}"
  }
}
```

#### OpenClaw
Status: Awaiting OpenClaw's skill system implementation. Track progress at https://github.com/OpenClaw/OpenClaw

### Platform Capability Matrix

| Feature | OpenCode | Claude Code | Claude Desktop | ChatGPT | n8n | OpenClaw |
|---------|----------|-------------|----------------|---------|-----|----------|
| Auto-trigger | ✅ | ❌ | ❌ | ❌ | ⚠️ | TBD |
| Git context | ✅ | ✅ | ⚠️ | ❌ | ⚠️ | TBD |
| File system | ✅ | ✅ | ✅ | ⚠️ | ✅ | TBD |
| Session tracking | ✅ | ✅ | ⚠️ | ❌ | ✅ | TBD |
| MCP servers | ✅ | ✅ | ✅ | ❌ | ⚠️ | TBD |
| Environment vars | ✅ | ✅ | ⚠️ | ❌ | ✅ | TBD |

Legend: ✅ Full | ⚠️ Limited | ❌ Not supported | TBD To be determined

## Auto-Trigger Behavior

When enabled in AGENTS.md, this skill will:

1. **On session start** - Check if current directory is a git repo
2. **Detect context** - Identify branch, recent commits, open MRs
3. **Create worklog** - Initialize worklog entry for the session
4. **Log intent** - Capture the user's first message as the topic

**Note:** Auto-trigger only available on OpenCode. Other platforms require manual invocation.

## AGENTS.md Integration

Add to your `AGENTS.md` file:

```markdown
# Auto Worklog Configuration

## Pre-Flight Checklist

Before starting work, the agent MUST:

1. Initialize worklog entry for this session
2. Log the topic/intent of work
3. Record current git context

## Worklog Path Configuration

### Cross-Platform Path Resolution

The skill uses a hierarchical path resolution strategy that works across platforms:

| Priority | Source | Description | Platform Support |
|----------|--------|-------------|------------------|
| 1 | `AUTO_WORKLOG_DIR` env var | User-defined worklog location | All platforms |
| 2 | Platform default | System-appropriate location | Platform-specific |
| 3 | `./worklogs/` | Project-local fallback | All platforms |

### Platform-Specific Defaults

```bash
# Linux/macOS (OpenCode, Claude Code, Claude Desktop)
export AUTO_WORKLOG_DIR="${HOME}/.local/share/ai-hub/worklog/"

# Windows (Claude Desktop)
set AUTO_WORKLOG_DIR=%USERPROFILE%\AppData\Local\ai-hub\worklog\

# n8n (Docker/containerized)
export AUTO_WORKLOG_DIR="/data/worklogs/"

# ChatGPT (API-based, configure via settings)
WORKLOG_DIR="/api/v1/worklogs/"
```

### Path Resolution Logic

```typescript
function resolveWorklogDir(): string {
  // Priority 1: Environment variable
  if (process.env.AUTO_WORKLOG_DIR) {
    return process.env.AUTO_WORKLOG_DIR;
  }
  
  // Priority 2: Platform-specific default
  const home = process.env.HOME || process.env.USERPROFILE;
  if (home) {
    return path.join(home, '.local', 'share', 'ai-hub', 'worklog');
  }
  
  // Priority 3: Fallback to current directory
  return './worklogs/';
}
```

## File Format

`YYYY-MM-DD {Topic}.md`
```

## Workflow

### Standard Session Flow

```
User: "Review this MR..."
↓
Agent detects work intent
↓
Auto-initializes worklog:
  - File: 2025-02-18 MR Review.md
  - Topic: Review MR !11
  - Context: Current repo, branch
↓
Agent responds: "📝 Worklog initialized: ~/repos/system/worklog/2025-02-18 MR Review.md"
↓
Agent proceeds with MR review
↓
Agent updates worklog with findings
```

## Detection Logic

The skill detects work intent from:

- Explicit requests: "work on...", "implement...", "fix..."
- Review requests: "review...", "check...", "analyze..."
- Refactor requests: "refactor...", "reorganize...", "clean up..."
- Debug requests: "debug...", "investigate...", "why is..."

## Worklog Sections (Auto-Populated)

```markdown
# Worklog: {Detected Topic}

**Session ID:** {SESSION_ID}  
**Date:** YYYY-MM-DD  
**Time:** HH:MM:SS  
**Timezone:** {TIMEZONE} (e.g., Australia/Brisbane)  
**Agent:** Sisyphus  
**Repository:** {REPO_NAME}  
**Branch:** {GIT_BRANCH}  
**Status:** 🟡 In Progress

## Prompt and User Feedback

### Initial Prompt
```
{Full user message as received}
```

### User Feedback/Responses
```
{Any clarifications, corrections, or feedback provided by user during session}
```

## Request Context

### User Request
{Full user message}

### Auto-Detected Context
- Working Directory: {PWD}
- Git Status: {STATUS}
- Recent Commits: {LAST_3_COMMITS}
- Open MRs: {MR_LIST}

## Skills Used

| Skill | Purpose | Invocation |
|-------|---------|------------|
| {skill_name} | {what it was used for} | {manual/auto} |

## MCP Servers Used

| Server | Tools Used | Purpose |
|--------|------------|---------|
| {server_name} | {tool1, tool2} | {purpose} |

## Context Used

### Files Referenced
- `{file_path}` - {reason/context}

### Sessions Referenced
- Session ID: {session_id} - {context}

### External Context
- Web sources: {URLs if fetched}
- Documentation: {docs referenced}

## Providers and Models Used

| Provider | Model | Purpose |
|----------|-------|---------|
| {provider_name} | {model_name} | {task_performed} |

## Agent Plan

{Agent's todo list / plan}

## Execution Log

- [HH:MM:SS] Session started
- [HH:MM:SS] Context gathered
- [HH:MM:SS] Work initiated

## To Be Updated...

- Files modified
- Decisions made
- Final summary
```

## Agent Instructions

When this skill is active:

1. **Always check first** - Is this a work session that needs logging?
2. **Create worklog early** - Before diving into code changes
3. **Update continuously** - Append to worklog as work progresses
4. **Final summary** - Complete worklog with final status

## Example Integration

```typescript
// Agent workflow with auto-worklog

// 1. User request comes in
userMessage = "Review MR !11"

// 2. Auto-detect work intent
if (isWorkRequest(userMessage)) {
  // 3. Initialize worklog with enhanced metadata
  worklogPath = createWorklog({
    topic: "MR Review !11",
    request: userMessage,
    userFeedback: "", // Populated during session
    context: gatherGitContext(),
    skillsUsed: [],
    mcpServersUsed: [],
    providersUsed: [],
    modelsUsed: [],
    timezone: getSystemTimezone(),
    worklogDir: process.env.AUTO_WORKLOG_DIR || 
                `${process.env.HOME}/.local/share/ai-hub/worklog/`
  })
  
  // 4. Inform user
  output(`📝 Worklog: ${worklogPath}`)
}

// 5. Proceed with actual work
// ... review MR ...

// 6. Update worklog with results and metadata
appendToWorklog(worklogPath, {
  files: changedFiles,
  summary: reviewResults,
  skillsUsed: ["mr-code-review", "git-master"],
  mcpServersUsed: ["github-mcp"],
  providersUsed: ["opencode"],
  modelsUsed: ["claude-haiku-4-5"]
})
```

## Platform-Specific Limitations & Workarounds

When using this skill across platforms, be aware of these limitations:

### Git Context Availability

| Platform | Git Branch | Recent Commits | Git Status |
|----------|-----------|----------------|------------|
| OpenCode | ✅ Full | ✅ Full | ✅ Full |
| Claude Code | ✅ Full | ✅ Full | ✅ Full |
| Claude Desktop | ⚠️ Via MCP | ⚠️ Via MCP | ❌ Limited |
| ChatGPT | ❌ N/A | ❌ N/A | ❌ N/A |
| n8n | ✅ Via node | ✅ Via node | ✅ Via node |

**Workaround for limited/no Git access:**
- Prompt user to manually provide branch name
- Use directory name as repository identifier
- Skip Git-dependent metadata sections

### Session Tracking

| Platform | Session ID | Context Persistence |
|----------|-----------|---------------------|
| OpenCode | ✅ Native | ✅ Full |
| Claude Code | ✅ Native | ✅ Full |
| Claude Desktop | ⚠️ Manual | ⚠️ Limited |
| ChatGPT | ❌ Per-message | ❌ None |
| n8n | ✅ Execution ID | ✅ Workflow-scoped |

**Workaround for missing session ID:**
- Generate UUID: `crypto.randomUUID()`
- Use timestamp + topic hash
- Omit session references

### Environment Variables

| Platform | Process.env | Platform Vars |
|----------|-------------|---------------|
| OpenCode | ✅ Full | ✅ Full |
| Claude Code | ✅ Full | ✅ Full |
| Claude Desktop | ⚠️ Limited | ⚠️ Limited |
| ChatGPT | ❌ N/A | ❌ N/A |
| n8n | ✅ Via $env | ✅ Full |

**Workaround:**
- Claude Desktop: Use MCP tool configuration
- ChatGPT: Use GPT Action parameters
- Provide manual configuration option

### File System Access

| Platform | Read | Write | Path Resolution |
|----------|------|-------|-----------------|
| OpenCode | ✅ Full | ✅ Full | ✅ Full |
| Claude Code | ✅ Full | ✅ Full | ✅ Full |
| Claude Desktop | ✅ Via MCP | ✅ Via MCP | ⚠️ Sandbox paths |
| ChatGPT | ⚠️ API only | ⚠️ API only | ⚠️ Virtual paths |
| n8n | ✅ Full | ✅ Full | ✅ Full |

**Workaround for restricted file access:**
- Use platform-provided file APIs
- Store in platform-specific locations
- Offer clipboard/export as alternative

## Metadata Capture Requirements

When creating worklog entries, agents MUST capture:

### Required Fields
- **Prompt**: Full user message as code block
- **Timezone**: System timezone (e.g., "Australia/Brisbane")
- **Date/Time**: ISO 8601 format with timezone

### Skills Tracking
Log every skill invoked during the session:
```markdown
## Skills Used

| Skill | Purpose | Invocation |
|-------|---------|------------|
| git-master | Commit and push changes | manual via skill() |
| mr-code-review | Review merge request | auto-triggered |
```

### MCP Servers Tracking
Document which MCP servers and tools were used:
```markdown
## MCP Servers Used

| Server | Tools Used | Purpose |
|--------|------------|---------|
| github-mcp | get_pr, post_comment | Fetch MR details |
| filesystem-mcp | read_file, write_file | File operations |
```

### Context Tracking
Record what context informed the work:
```markdown
## Context Used

### Files Referenced
- `/src/config.ts` - Configuration patterns
- `/docs/api.md` - API documentation

### Sessions Referenced
- Session ID: `ses_abc123` - Previous related work

### External Context
- Web: https://docs.example.com/api
- Docs: Context7 - Next.js v14 routing
```

### Providers and Models
Track AI providers and models used:
```markdown
## Providers and Models Used

| Provider | Model | Purpose |
|----------|-------|---------|
| opencode | claude-haiku-4-5 | Primary orchestration |
| openai | gpt-4 | Code generation |
| ollama | qwen2.5-coder-14b | Local inference |
```

## Best Practices

1. **Create on intent detection** - Don't wait for explicit command
2. **Update in real-time** - Append as work happens, not just at end
3. **Capture decisions** - Why did we do X instead of Y?
4. **Link everything** - MR URLs, commit hashes, file paths
5. **Note interruptions** - If work pauses, log why

## File Management

### Naming Collision Handling

If file exists:
```
2025-02-18 MR Review.md        # Original
2025-02-18 MR Review 2.md      # Collision 1
2025-02-18 MR Review 3.md      # Collision 2
```

### Archive Strategy

Old worklogs (90+ days) auto-move to:
`${AUTO_WORKLOG_DIR}/archive/YYYY/MM/`

Default path: `~/.local/share/ai-hub/worklog/archive/YYYY/MM/`

## Cross-Platform Migration

### Export/Import Worklogs

If moving worklogs between platforms:

```bash
# Export from OpenCode/Claude Code
export AUTO_WORKLOG_DIR="$HOME/.local/share/ai-hub/worklog/"
tar -czf worklogs-backup.tar.gz "$AUTO_WORKLOG_DIR"

# Import to n8n
# Upload to n8n file storage or mount volume

# Import to Claude Desktop
# Use MCP file system tools to write to configured path
```

### Platform-Specific Path Mapping

When migrating, update paths in worklog headers:

| From Platform | To Platform | Path Translation |
|--------------|-------------|------------------|
| OpenCode (Linux) | Claude Desktop (macOS) | `~/.local/share/ai-hub/` → `~/Library/Application Support/ai-hub/` |
| Claude Code | n8n | Use absolute paths in container |
| ChatGPT | Any | Re-export via API download |

### Preserving Metadata

Some metadata may be platform-specific:
- **Session IDs**: Not transferable, keep for reference
- **Git context**: Re-resolve on new platform
- **Environment vars**: Re-document with new values
- **Provider/Model**: Update to reflect current platform's models

## Rules

- ALWAYS create worklog before making file changes
- NEVER skip logging for "quick" tasks
- Update worklog when work pauses or completes
- Include timestamps for all significant events
- Link to external resources (MRs, issues, docs)
- CAPTURE timezone for all timestamps
- LOG all skills, MCP servers, and models used
- RECORD user feedback inline as code blocks
- TRACK all context sources (files, sessions, web)
- **CONSIDER PLATFORM LIMITATIONS** - Adapt metadata capture for platform capabilities
- **DOCUMENT PLATFORM USED** - Note which platform generated the worklog
- **HANDLE MISSING CONTEXT** - Gracefully skip unavailable context (Git, env vars, etc.)

---

## Attribution

**Skill Author:** Gavin Tomlins (gavintomlins+ai-hub@gmail.com)  
**Repository:** https://github.com/GavinTomlins/ai-hub  
**License:** Apache 2.0

### License Notice

```
Copyright 2025 Gavin Tomlins

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

## Worklog Template Footer

All worklog files should include this footer:

```markdown
---

**Worklog Generated by:** auto-worklog skill  
**Author:** Gavin Tomlins  
**Repository:** https://github.com/GavinTomlins/ai-hub  
**License:** Apache 2.0  
*Last Updated: {TIMESTAMP} | Timezone: {TIMEZONE}*
```
