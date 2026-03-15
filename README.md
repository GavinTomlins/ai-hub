# AI-Hub

**Cross-Platform AI Agent Skills and Integrations**

A collection of portable, platform-agnostic skills for AI agents across OpenCode, Claude Code, Claude Desktop, ChatGPT, n8n, and OpenClaw.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

## Author

**Gavin Tomlins**  
📧 gavintomlins+ai-hub@gmail.com  
🔗 https://github.com/GavinTomlins/ai-hub

## Related Projects & References

This repository integrates with and builds upon the following open-source projects:

| Project | Description | Repository | Documentation |
|---------|-------------|------------|---------------|
| **OpenCode** | AI coding agent for the terminal | [anomalyco/opencode](https://github.com/anomalyco/opencode) | [OpenCode Docs](https://github.com/anomalyco/opencode) |
| **Oh-My-OpenCode** | Agent harness and plugin system for OpenCode | [code-yeongyu/oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent) | [OMO Docs](https://github.com/code-yeongyu/oh-my-openagent) |
| **Graphiti** | Real-time knowledge graphs for AI agents | [getzep/graphiti](https://github.com/getzep/graphiti) | [Graphiti Docs](https://help.getzep.com/graphiti) |
| **FalkorDB** | Low-latency graph database (Redis-compatible) | [FalkorDB/falkordb](https://github.com/FalkorDB/falkordb) | [FalkorDB Docs](https://docs.falkordb.com) |
| **Neo4j** | Native graph database with Cypher query language | [neo4j/neo4j](https://github.com/neo4j/neo4j) | [Neo4j Docs](https://neo4j.com/docs/) |

*AI-Hub provides pre-configured skills and integration patterns for these technologies, tested in production environments.*

## Overview

This repository contains production-ready skills that work across multiple AI platforms. Each skill is designed with:

- ✅ **Cross-platform compatibility** - Works on OpenCode, Claude, ChatGPT, n8n, and more
- ✅ **Platform-agnostic paths** - No hardcoded directories
- ✅ **Graceful degradation** - Adapts to platform capabilities
- ✅ **Comprehensive documentation** - Setup guides for each platform
- ✅ **Apache 2.0 License** - Free for personal and commercial use

## Quick Start

### For OpenCode Users

1. **Clone the repository:**
   ```bash
   git clone https://github.com/GavinTomlins/ai-hub.git ~/repos/personal/ai-hub
   ```

2. **Add to your AGENTS.md:**
   ```markdown
   ## Startup Behavior
   
   ### Pre-Flight Checklist
   
   Before starting work, the agent MUST:
   
   1. Initialize worklog entry for this session
   2. Log the topic/intent of work
   3. Record current git context
   
   ### Auto-Trigger Agents
   
   | Agent | Purpose | Auto-Trigger |
   |-------|---------|--------------|
   | **worklogger** | Automatic worklog tracking | Yes - On work intent detection |
   
   ## Worklog Configuration
   
   **Path:** Set via `AUTO_WORKLOG_DIR` environment variable  
   **Default:** `~/.local/share/ai-hub/worklog/`
   
   ## Detection Patterns
   
   Work intent is detected from:
   - Explicit requests: "work on...", "implement...", "fix..."
   - Review requests: "review...", "check...", "analyze..."
   - Refactor requests: "refactor...", "reorganize...", "clean up..."
   - Debug requests: "debug...", "investigate...", "why is..."
   - Plan requests: "plan...", "design...", "architecture..."
   ```

3. **Set environment variable (optional):**
   ```bash
   # Add to ~/.zshrc or ~/.bashrc
   export AUTO_WORKLOG_DIR="$HOME/.local/share/ai-hub/worklog/"
   ```

### For Other Platforms

See individual skill documentation for platform-specific setup:

- [Claude Code Setup](.agents/skills/auto-worklog/SKILL.md#claude-code)
- [Claude Desktop Setup](.agents/skills/auto-worklog/SKILL.md#claude-desktop)
- [ChatGPT Setup](.agents/skills/auto-worklog/SKILL.md#chatgpt)
- [n8n Setup](.agents/skills/auto-worklog/SKILL.md#n8n)
- [OpenClaw Status](.agents/skills/auto-worklog/SKILL.md#openclaw)

## Available Skills

### auto-worklog ⭐ REQUIRED

**AUTOMATIC worklog initialization** - Creates worklog entry at start of every session.

- **Auto-Trigger:** Yes (OpenCode) / Manual (other platforms)
- **Platforms:** OpenCode ✅, Claude Code ⚠️, Claude Desktop ⚠️, ChatGPT ⚠️, n8n ⚠️, OpenClaw 🧪
- **Features:**
  - Automatic worklog creation on work intent detection
  - Git context capture (branch, commits, status)
  - Skills, MCP servers, and models tracking
  - Prompt and user feedback logging
  - Timezone-aware timestamps
  - Cross-platform path resolution
- **Quick Start:** [Installation Guide](#installation) → [OpenCode Integration](docs/OPENCODE_INTEGRATION.md)

[View Documentation →](.agents/skills/auto-worklog/SKILL.md)

### graphiti-memory (Node Sage) 🧠 OPTIONAL

**Persistent memory storage** - Enables cross-session context and team knowledge sharing.

- **Auto-Trigger:** Optional (when MCP server available)
- **Platforms:** OpenCode ✅, Claude Code ⚠️, Claude Desktop ⚠️, ChatGPT ❌, n8n 🧪, OpenClaw 🧪
- **Requirements:** MCP server (Neo4j or FalkorDB backend)
- **Features:**
  - Cross-session memory persistence
  - Semantic search via graph database
  - Team knowledge sharing
  - Integration with auto-worklog for enriched context
- **Best For:** Long-running projects, team collaboration, complex context
- **Setup Time:** 10-15 minutes (MCP server deployment)
- **Quick Start:** [MCP Setup Guide](docs/NODE_SAGE_INTEGRATION.md#mcp-server-setup) → [OpenCode Configuration](docs/NODE_SAGE_INTEGRATION.md#opencode-configuration)

[View Documentation →](.agents/skills/graphiti-memory/SKILL.md) | [Setup Guide →](docs/NODE_SAGE_INTEGRATION.md)

### Skill Relationships

```
auto-worklog (REQUIRED)
├── Creates worklog on every session
├── Tracks work progress
└── Works standalone

graphiti-memory (OPTIONAL)
├── Provides persistent memory
├── Enriches auto-worklog context
└── Requires MCP server

Recommended: Start with auto-worklog, add Node Sage later
```

## Repository Structure

```
ai-hub/
├── .agents/
│   └── skills/
│       ├── auto-worklog/         # Required: Automatic worklog
│       │   └── SKILL.md
│       └── graphiti-memory/      # Optional: Persistent memory
│           └── SKILL.md
├── docs/
│   ├── OPENCODE_INTEGRATION.md   # OpenCode setup guide
│   └── NODE_SAGE_INTEGRATION.md  # Node Sage MCP setup
├── .gitignore                    # Git ignore rules
├── LICENSE                       # Apache 2.0 License
└── README.md                     # This file
```

## Documentation Index

| Document | Purpose | Skill |
|----------|---------|-------|
| [README.md](README.md) | Overview and quick start | All |
| [SKILL.md](.agents/skills/auto-worklog/SKILL.md) | auto-worklog reference | auto-worklog |
| [SKILL.md](.agents/skills/graphiti-memory/SKILL.md) | Node Sage reference | graphiti-memory |
| [OPENCODE_INTEGRATION.md](docs/OPENCODE_INTEGRATION.md) | OpenCode installation | auto-worklog |
| [NODE_SAGE_INTEGRATION.md](docs/NODE_SAGE_INTEGRATION.md) | MCP server setup | graphiti-memory |

## Installation

### Option 1: Clone to Recommended Location

```bash
# Create directory structure
mkdir -p ~/repos/personal
cd ~/repos/personal

# Clone repository
git clone https://github.com/GavinTomlins/ai-hub.git
```

### Option 2: Install Individual Skills (Recommended)

**Architecture:** Skills are stored in `~/.agents/skills/` (tool-agnostic location), and OpenCode symlinks to this location.

```bash
# Step 1: Create the canonical skills directory
mkdir -p ~/.agents/skills

# Step 2: Symlink the skill from ai-hub to canonical location
ln -s ~/repos/personal/ai-hub/.agents/skills/auto-worklog \
  ~/.agents/skills/auto-worklog

# Step 3: Ensure OpenCode skills directory points to canonical location
# Check if ~/.config/opencode/skills exists
ls -la ~/.config/opencode/skills 2>/dev/null && echo "exists" || echo "does not exist"

# If it doesn't exist or is a regular directory, replace with symlink:
# (Backup any existing skills first!)
mv ~/.config/opencode/skills ~/.config/opencode/skills.backup 2>/dev/null
ln -s ~/.agents/skills ~/.config/opencode/skills
```

**Why this structure?**
- `~/.agents/skills/` - Tool-agnostic canonical location
- `~/.config/opencode/skills` - Symlink to canonical location
- Allows sharing skills across different tools (OpenCode, Claude Desktop, etc.)

## Oh-My-OpenCode Integration

If you're using [oh-my-opencode](https://github.com/opencode/oh-my-opencode), integrate this skill by:

1. **Set up the skills directory structure:**
   ```bash
   # Create canonical skills directory (tool-agnostic)
   mkdir -p ~/.agents/skills
   
   # Symlink skill from ai-hub to canonical location
   ln -s ~/repos/personal/ai-hub/.agents/skills/auto-worklog \
     ~/.agents/skills/auto-worklog
   
   # Point OpenCode to the canonical skills directory
   # (Backup existing if needed, then create symlink)
   mv ~/.config/opencode/skills ~/.config/opencode/skills.backup.$(date +%Y%m%d) 2>/dev/null
   ln -s ~/.agents/skills ~/.config/opencode/skills
   ```
   
   **Directory Structure:**
   ```
   ~/.agents/skills/              # Canonical skills location (tool-agnostic)
   └── auto-worklog/             # Actual skill directory
       └── SKILL.md
   
   ~/.config/opencode/skills -> ~/.agents/skills  # Symlink
   ```

2. **Update your AGENTS.md** to include worklogger in startup:
   ```markdown
   ### Startup Behavior (Every Prompt)
   
   On every user interaction, the following agents are automatically invoked:
   
   1. **Node Sage** (`graphiti-memory`) - Initializes memory context
   2. **worklogger** - Initializes/updates worklog for current session
      - Detects work intent from user message
      - Creates worklog entry with timestamp and context
      - Appends to existing worklog if same day/topic
   
   ### Worklogger Configuration
   
   **Trigger:** Every user message indicating work intent
   
   **Detection patterns:**
   - Explicit: "work on...", "implement...", "fix..."
   - Review: "review...", "check...", "analyze..."
   - Refactor: "refactor...", "reorganize...", "clean up..."
   - Debug: "debug...", "investigate...", "why is..."
   - Plan: "plan...", "design...", "architecture..."
   
   **Environment:**
   - `AUTO_WORKLOG_DIR` - Custom worklog location
   - Default: `~/.local/share/ai-hub/worklog/`
   ```

3. **Create the worklog directory:**
   ```bash
   mkdir -p ~/.local/share/ai-hub/worklog/
   ```

### Optional: Add Node Sage (Enhanced Context)

For persistent memory across sessions, add Node Sage:

1. **Set up MCP Server** (10-15 minutes):
   ```bash
   # Follow detailed setup guide
   # See: docs/NODE_SAGE_INTEGRATION.md
   ```

2. **Add Node Sage to AGENTS.md** (Optional):
   ```markdown
   ### Startup Behavior (Every Prompt)
   
   On every user interaction:
   
   1. **worklogger** (REQUIRED) - Creates worklog entry
      - Auto-triggered on work intent detection
      - Source: auto-worklog skill
   
   2. **Node Sage** (OPTIONAL) - Loads persistent memory
      - Available when MCP server running
      - Falls back gracefully if unavailable
      - Source: graphiti-memory skill
      - Enriches worklog with cross-session context
   ```

3. **Configure Environment:**
   ```bash
   # Required for Node Sage
   export OPENAI_API_KEY="sk-your-api-key"
   
   # Optional for Neo4j
   export NEO4J_PASSWORD="secure-password"
   ```

[Full Node Sage Setup Guide →](docs/NODE_SAGE_INTEGRATION.md)

## Configuration

### Environment Variables

#### auto-worklog

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `AUTO_WORKLOG_DIR` | Worklog storage directory | `~/.local/share/ai-hub/worklog/` | No |

#### graphiti-memory (Node Sage) - Optional

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `OPENAI_API_KEY` | OpenAI API key for embeddings | - | **Yes** (for MCP) |
| `NEO4J_PASSWORD` | Neo4j database password | `graphiti` | No |

### Platform-Specific Paths

| Platform | Default Path |
|----------|-------------|
| Linux/macOS | `~/.local/share/ai-hub/worklog/` |
| Windows | `%USERPROFILE%\AppData\Local\ai-hub\worklog\` |
| n8n (Docker) | `/data/worklogs/` |
| ChatGPT | API endpoint configured in GPT |

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-skill`
3. Follow the [skill structure guidelines](CONTRIBUTING.md)
4. Ensure cross-platform compatibility
5. Add platform-specific setup instructions
6. Submit a pull request

## License

Apache 2.0 - See [LICENSE](LICENSE) for details.

```
Copyright 2025 Gavin Tomlins

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0
```

## Support

- 📧 Email: gavintomlins+ai-hub@gmail.com
- 🐛 Issues: [GitHub Issues](https://github.com/GavinTomlins/ai-hub/issues)
- 💬 Discussions: [GitHub Discussions](https://github.com/GavinTomlins/ai-hub/discussions)

## Roadmap

- [x] auto-worklog skill with cross-platform support
- [x] graphiti-memory (Node Sage) skill with MCP integration
- [x] Comprehensive documentation and setup guides
- [ ] Additional cross-platform skills
- [ ] MCP server implementations
- [ ] Automated testing across platforms
- [ ] Video tutorials for each platform

---

## Quick Reference

**Get started in 5 minutes:**
```bash
# 1. Clone repository
git clone https://github.com/GavinTomlins/ai-hub.git ~/repos/personal/ai-hub

# 2. Install auto-worklog (REQUIRED)
mkdir -p ~/.agents/skills
ln -s ~/repos/personal/ai-hub/.agents/skills/auto-worklog ~/.agents/skills/
ln -s ~/.agents/skills ~/.config/opencode/skills

# 3. Create worklog directory
mkdir -p ~/.local/share/ai-hub/worklog/

# 4. Update AGENTS.md (see docs/OPENCODE_INTEGRATION.md)

# 5. Done! Test with: "Work on my project"
```

**Add Node Sage later (optional):**
```bash
# Follow detailed guide
# See: docs/NODE_SAGE_INTEGRATION.md
```

---

**Made with ❤️ for the AI agent community**
