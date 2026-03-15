---
name: graphiti-memory
description: Persistent memory storage and retrieval using Graphiti graph database. Enable agents to remember context across sessions and share knowledge between team members. Optional companion to auto-worklog for enhanced context awareness.
author: Gavin Tomlins
email: gavintomlins+ai-hub@gmail.com
repository: https://github.com/GavinTomlins/ai-hub
license: Apache-2.0
compatibility:
  platforms:
    opencode:
      status: full
      auto_trigger: true
      notes: Native MCP server integration via opencode.json
    claude_code:
      status: partial
      auto_trigger: false
      notes: Requires manual MCP server setup
    claude_desktop:
      status: partial
      auto_trigger: false
      notes: Requires MCP server configuration in settings
    chatgpt:
      status: not_supported
      auto_trigger: false
      notes: Requires external API integration (not yet available)
    n8n:
      status: experimental
      auto_trigger: false
      notes: Via HTTP Request node to MCP server
    openclaw:
      status: experimental
      auto_trigger: false
      notes: Pending OpenClaw MCP support
  min_opencode_version: "0.5.0"
  requires_mcp_server: true
metadata:
  audience: developers
  workflow: memory-management
  auto-trigger: optional
  related_skills:
    - auto-worklog
---

# Node Sage (graphiti-memory)

**Persistent Memory for AI Agents**

Node Sage provides durable memory storage using Graphiti, a graph database optimized for AI agent context. It enables agents to remember information across sessions and share knowledge between team members.

**Author:** Gavin Tomlins (gavintomlins+ai-hub@gmail.com)  
**Repository:** https://github.com/GavinTomlins/ai-hub  
**License:** Apache 2.0

## What is Node Sage?

Node Sage is the friendly name for the `graphiti-memory` skill. It pairs with a local Graphiti MCP (Model Context Protocol) server to provide:

- 🧠 **Cross-session memory** - Remember context from previous conversations
- 🤝 **Team knowledge sharing** - Multiple agents can access shared memory
- 🔍 **Semantic search** - Find relevant facts using natural language queries
- 📊 **Graph relationships** - Understand connections between concepts
- 🏷️ **Organized storage** - Group memory by user, project, or context

## Architecture Overview

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│   AI Agent      │────▶│  MCP Server      │────▶│  Graph Database │
│  (OpenCode)     │◄────│  (Graphiti)      │◄────│ (Neo4j/FalkorDB)│
└─────────────────┘     └──────────────────┘     └─────────────────┘
        │
        │ Uses skill("graphiti-memory")
        ▼
┌─────────────────┐
│  Node Sage      │
│  (This Skill)   │
└─────────────────┘
```

## MCP Server Requirements

This skill requires a running Graphiti MCP server. Two backend options are available:

### Option 1: Neo4j Backend (Recommended)

**Best for:** Complex queries, production use, large knowledge graphs

**Services:**
- Neo4j graph database (port 7474/7687)
- Graphiti MCP server (port 8000)

### Option 2: FalkorDB Backend

**Best for:** Lightweight deployment, Redis-compatible operations

**Services:**
- FalkorDB (port 6379)
- Graphiti MCP server (port 8001)

## MCP Server Setup

### Prerequisites

- Docker and Docker Compose installed
- OpenAI API key (for embeddings)
- 4GB RAM minimum (8GB recommended)

### Quick Start with Docker Compose

#### Step 1: Create Docker Compose File

Create `~/ai-hub/graphiti/docker-compose.yml`:

```yaml
services:
  graph:
    image: graphiti/graphiti-mcp:latest
    ports:
      - "8000:8000"
    healthcheck:
      test: ["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:8000/healthcheck')"]
      interval: 10s
      timeout: 5s
      retries: 3
    depends_on:
      neo4j:
        condition: service_healthy
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - NEO4J_URI=bolt://neo4j:7687
      - NEO4J_USER=neo4j
      - NEO4J_PASSWORD=${NEO4J_PASSWORD:-graphiti}
      - PORT=8000
      - db_backend=neo4j

  neo4j:
    image: neo4j:5.26.2
    healthcheck:
      test: ["CMD-SHELL", "wget -qO- http://localhost:7474 || exit 1"]
      interval: 1s
      timeout: 10s
      retries: 10
      start_period: 3s
    ports:
      - "7474:7474"  # HTTP browser
      - "7687:7687"  # Bolt protocol
    volumes:
      - neo4j_data:/data
    environment:
      - NEO4J_AUTH=neo4j/${NEO4J_PASSWORD:-graphiti}

volumes:
  neo4j_data:
```

#### Step 2: Set Environment Variables

```bash
# Add to ~/.zshrc or ~/.bashrc
export OPENAI_API_KEY="sk-your-openai-api-key"
export NEO4J_PASSWORD="your-secure-password"  # Optional, defaults to 'graphiti'

# Reload shell configuration
source ~/.zshrc  # or source ~/.bashrc
```

#### Step 3: Start the MCP Server

```bash
cd ~/ai-hub/graphiti
docker-compose up -d

# Wait for services to be healthy (about 30-60 seconds)
docker-compose ps

# View logs
docker-compose logs -f graph
```

#### Step 4: Verify MCP Server is Running

```bash
# Check health endpoint
curl http://127.0.0.1:8000/health

# Expected output:
# {"status": "healthy"}
```

### Alternative: FalkorDB Backend

For a lighter-weight option using FalkorDB (Redis-compatible):

```yaml
services:
  falkordb:
    image: falkordb/falkordb:latest
    ports:
      - "6379:6379"
    volumes:
      - falkordb_data:/data
    environment:
      - FALKORDB_ARGS=--port 6379 --cluster-enabled no
    healthcheck:
      test: ["CMD", "redis-cli", "-p", "6379", "ping"]
      interval: 1s
      timeout: 10s
      retries: 10
      start_period: 3s

  graph-falkordb:
    image: graphiti/graphiti-mcp:latest
    ports:
      - "8001:8001"
    depends_on:
      falkordb:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:8001/healthcheck')"]
      interval: 10s
      timeout: 5s
      retries: 3
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - FALKORDB_HOST=falkordb
      - FALKORDB_PORT=6379
      - FALKORDB_DATABASE=default_db
      - GRAPHITI_BACKEND=falkordb
      - PORT=8001
      - db_backend=falkordb

volumes:
  falkordb_data:
```

Start with: `docker-compose up -d`

## OpenCode Configuration

### Step 1: Configure MCP Server in opencode.json

Edit `~/.config/opencode/opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "graphiti-memory": {
      "command": [
        "npx",
        "-y",
        "mcp-remote",
        "http://127.0.0.1:8000/sse",
        "--allow-http"
      ],
      "enabled": true,
      "type": "local"
    }
  },
  "skills": {
    "graphiti-memory": {
      "path": "~/.agents/skills/graphiti-memory",
      "type": "local"
    },
    "node-sage": {
      "path": "~/.agents/skills/graphiti-memory",
      "type": "local"
    }
  }
}
```

### Step 2: Configure Agent Prompts (Optional)

If using oh-my-opencode, edit `~/.config/opencode/oh-my-opencode.json`:

```json
{
  "agents": {
    "sisyphus": {
      "model": "kimi/kimi-k2.5-coding",
      "prompt": "Load Node Sage memory context first via skill(\"graphiti-memory\"). Use the name \"Node Sage\" in user-facing text."
    },
    "explore": {
      "model": "kimi/kimi-k2.5-coding",
      "prompt": "Load Node Sage memory context first via skill(\"graphiti-memory\"). Use the name \"Node Sage\" in user-facing text."
    }
  }
}
```

### Step 3: Install the Skill

```bash
# Create canonical skills directory
mkdir -p ~/.agents/skills

# Symlink from ai-hub repository
ln -s ~/repos/personal/ai-hub/.agents/skills/graphiti-memory \
  ~/.agents/skills/graphiti-memory

# Ensure OpenCode points to canonical location
ln -s ~/.agents/skills ~/.config/opencode/skills
```

## Usage

### Basic Memory Operations

#### Store Memory

```typescript
// Add a new memory episode
add_memory(
  name="Project Architecture Decision",
  episode_body="We decided to use PostgreSQL for the main database and Redis for caching. This provides ACID compliance for critical data and fast access for session storage.",
  group_id="project:myapp",
  source="text",
  source_description="architecture meeting notes"
)
```

#### Search Memory

```typescript
// Search for relevant memory nodes
search_memory_nodes(
  query="database decisions PostgreSQL",
  group_ids=["project:myapp", "user:username"],
  max_nodes=10
)

// Search for specific facts
search_memory_facts(
  query="caching strategy",
  group_ids=["project:myapp"],
  max_facts=10
)
```

#### Retrieve History

```typescript
// Get all episodes for a group
get_episodes(
  group_ids=["project:myapp"],
  max_episodes=50
)
```

### Group ID Convention

Always use explicit group IDs for stable cross-platform memory:

| Group Type | Format | Example |
|------------|--------|---------|
| User memory | `user:<username>` | `user:gavintomlins` |
| Project memory | `project:<name>` | `project:ai-hub` |
| Temporary | `opencode-local` | `opencode-local` |

**Important:** Do not derive project identity from local filesystem paths.

### Integration with auto-worklog

Node Sage works as an optional companion to auto-worklog:

```typescript
// At session start, load context
skill("graphiti-memory")

// Search for related work
search_memory_facts(
  query="recent work on authentication",
  group_ids=["project:myapp", "user:gavintomlins"]
)

// After completing work, store insights
add_memory(
  name="Authentication Implementation",
  episode_body="Implemented JWT-based auth with refresh tokens. Key decisions: 1) Access tokens expire in 15min, 2) Refresh tokens valid for 7 days, 3) Using httpOnly cookies for security.",
  group_id="project:myapp"
)
```

## Available MCP Tools

When the MCP server is running, these tools become available:

| Tool | Purpose | Parameters |
|------|---------|------------|
| `add_memory` | Store new memory | `name`, `episode_body`, `group_id`, `source` |
| `search_memory_nodes` | Find memory nodes | `query`, `group_ids`, `max_nodes` |
| `search_memory_facts` | Search for facts | `query`, `group_ids`, `max_facts` |
| `get_episodes` | Retrieve episode history | `group_ids`, `max_episodes` |
| `delete_episode` | Remove an episode | `uuid` |
| `clear_graph` | Clear all data | `group_ids` |
| `get_status` | Check server status | - |

## Best Practices

### Memory Storage

1. **Be specific** - Store concrete facts, not vague statements
2. **Keep it small** - Break large concepts into atomic facts
3. **Use consistent group IDs** - Don't change group naming schemes
4. **Search before adding** - Check if information already exists
5. **Never store secrets** - No passwords, API keys, or credentials

### Memory Retrieval

1. **Load context first** - Always call skill at session start
2. **Use multiple group IDs** - Search across user and project memory
3. **Handle missing context** - Gracefully continue if no memory found
4. **Update continuously** - Add new facts as work progresses

### Examples of Good Memory

✅ **Good:** "PostgreSQL 15 with pgvector extension for embeddings"  
✅ **Good:** "JWT tokens expire in 15 minutes, refresh in 7 days"  
✅ **Good:** "API rate limit: 1000 requests/hour per API key"  

❌ **Bad:** "We did some database stuff"  
❌ **Bad:** "Password: mysecret123"  
❌ **Bad:** "File located at /Users/john/projects/..."

## Endpoints Reference

Once the MCP server is running:

| Endpoint | URL | Purpose |
|----------|-----|---------|
| MCP SSE | `http://127.0.0.1:8000/sse` | Model Context Protocol stream |
| Health | `http://127.0.0.1:8000/health` | Server health check |
| Neo4j Browser | `http://127.0.0.1:7474` | Graph visualization (Neo4j only) |

## Troubleshooting

### MCP Server Not Responding

```bash
# Check if containers are running
docker-compose ps

# View logs
docker-compose logs graph

# Restart services
docker-compose restart

# Check port availability
lsof -i :8000
```

### OpenCode Can't Connect to MCP

```bash
# Verify MCP server is accessible
curl http://127.0.0.1:8000/health

# Check opencode.json syntax
npx jsonlint ~/.config/opencode/opencode.json

# Restart OpenCode after config changes
```

### Memory Not Persisting

1. Check group IDs are consistent
2. Verify MCP server has write access to database
3. Check database volume mounts in docker-compose.yml
4. Review MCP server logs for errors

### High Memory Usage

- Neo4j typically uses 2-4GB RAM
- FalkorDB uses less (~500MB)
- Consider FalkorDB for resource-constrained environments

## Platform-Specific Notes

### OpenCode (Full Support)

✅ Native MCP integration  
✅ Auto-trigger on session start  
✅ Agent prompt configuration  

### Claude Code (Partial Support)

⚠️ Requires manual MCP setup  
⚠️ No auto-trigger capability  

### Claude Desktop (Partial Support)

⚠️ Requires MCP configuration in settings  
⚠️ Limited to one MCP server at a time  

### ChatGPT (Not Supported)

❌ No native MCP support  
❌ Requires custom API integration  

## Maintenance

### Backup Memory

```bash
# Neo4j backup
docker exec -it graphiti-neo4j-1 neo4j-admin dump --to=/backups/backup.dump

# Or copy volume
docker run --rm -v graphiti_neo4j_data:/data -v ~/backups:/backups alpine tar czf /backups/neo4j-backup.tar.gz -C /data .
```

### Update MCP Server

```bash
docker-compose pull
docker-compose up -d
```

### Reset Memory

```bash
# Clear all data (WARNING: Destructive!)
docker-compose down -v
docker-compose up -d
```

## Integration Checklist

- [ ] Docker and Docker Compose installed
- [ ] OpenAI API key configured
- [ ] MCP server running (port 8000 accessible)
- [ ] opencode.json updated with MCP configuration
- [ ] Skill installed in `~/.agents/skills/`
- [ ] OpenCode skills directory points to canonical location
- [ ] Agent prompts configured (optional)
- [ ] Test memory operations successful
- [ ] Backup strategy in place

## Related Skills

- **auto-worklog** - Automatic work logging (companion skill)
- **pr-report** - PR analysis with memory integration
- **paperclip** - Task management with context awareness

## Rules

- ALWAYS initialize Node Sage at session start when needed
- NEVER store secrets, passwords, or credentials
- USE consistent group IDs across sessions
- SEARCH before adding to avoid duplicates
- KEEP memory entries atomic and specific
- HANDLE missing context gracefully
- UPDATE memory continuously as work progresses
- BACKUP regularly to prevent data loss

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

## Resources

- **Graphiti Documentation:** https://github.com/getzep/graphiti
- **Neo4j Documentation:** https://neo4j.com/docs/
- **FalkorDB Documentation:** https://docs.falkordb.com/
- **MCP Specification:** https://modelcontextprotocol.io/

*Last Updated: 2025-03-15 | Timezone: Australia/Brisbane*
