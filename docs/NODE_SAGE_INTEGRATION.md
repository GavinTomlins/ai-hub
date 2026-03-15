# Node Sage Integration Guide

**Setup and Configuration for Graphiti Memory (Node Sage)**

This guide provides detailed instructions for configuring the Node Sage memory system using Graphiti.

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [MCP Server Setup](#mcp-server-setup)
4. [OpenCode Configuration](#opencode-configuration)
5. [Integration with auto-worklog](#integration-with-auto-worklog)
6. [Verification](#verification)
7. [Troubleshooting](#troubleshooting)

## Overview

Node Sage (graphiti-memory skill) requires:

1. **Graphiti MCP Server** - Running locally via Docker
2. **Graph Database** - Neo4j or FalkorDB for storage
3. **OpenCode Configuration** - MCP client settings in `opencode.json`
4. **Skill Installation** - Symlinked to `~/.agents/skills/`

## Prerequisites

Before starting, ensure you have:

- [ ] Docker Engine 20.10+ and Docker Compose 2.0+
- [ ] OpenAI API key (for embeddings)
- [ ] OpenCode 0.5.0 or later
- [ ] 4GB available RAM (8GB recommended)
- [ ] Ports 8000, 7474, 7687 available (or configure alternatives)

### Check Prerequisites

```bash
# Docker version
docker --version
docker-compose --version

# Port availability
lsof -i :8000 2>/dev/null || echo "Port 8000 available"
lsof -i :7474 2>/dev/null || echo "Port 7474 available"

# OpenCode version
opencode --version  # or check your installation method
```

## MCP Server Setup

The Model Context Protocol (MCP) server acts as a bridge between OpenCode and the graph database.

### Step 1: Create MCP Server Directory

```bash
mkdir -p ~/ai-hub/graphiti
cd ~/ai-hub/graphiti
```

### Step 2: Create Docker Compose Configuration

Choose your backend:

#### Option A: Neo4j Backend (Recommended)

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  neo4j:
    image: neo4j:5.26.0
    container_name: graphiti-neo4j
    healthcheck:
      test: ["CMD", "wget", "-O", "/dev/null", "http://localhost:7474"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    ports:
      - "7474:7474"  # HTTP browser
      - "7687:7687"  # Bolt protocol
    volumes:
      - neo4j_data:/data
      - neo4j_logs:/logs
    environment:
      - NEO4J_AUTH=${NEO4J_USER:-neo4j}/${NEO4J_PASSWORD:-demodemo}
      - NEO4J_server_memory_heap_initial__size=512m
      - NEO4J_server_memory_heap_max__size=1G
      - NEO4J_server_memory_pagecache_size=512m
    networks:
      - graphiti-network

  graphiti-mcp:
    image: zepai/knowledge-graph-mcp:standalone
    container_name: graphiti-mcp
    env_file:
      - path: .env
        required: false
    depends_on:
      neo4j:
        condition: service_healthy
    environment:
      - NEO4J_URI=${NEO4J_URI:-bolt://neo4j:7687}
      - NEO4J_USER=${NEO4J_USER:-neo4j}
      - NEO4J_PASSWORD=${NEO4J_PASSWORD:-demodemo}
      - NEO4J_DATABASE=${NEO4J_DATABASE:-neo4j}
      - GRAPHITI_GROUP_ID=${GRAPHITI_GROUP_ID:-main}
      - SEMAPHORE_LIMIT=${SEMAPHORE_LIMIT:-10}
      - CONFIG_PATH=/app/mcp/config/config.yaml
      - PATH=/root/.local/bin:${PATH}
    ports:
      - "8000:8000"  # MCP server HTTP transport
    command: ["uv", "run", "main.py"]
    networks:
      - graphiti-network

volumes:
  neo4j_data:
  neo4j_logs:

networks:
  graphiti-network:
    driver: bridge
```

#### Option B: FalkorDB Backend (Lightweight)

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  falkordb:
    image: falkordb/falkordb:latest
    container_name: graphiti-falkordb
    ports:
      - "6379:6379"  # Redis/FalkorDB port
      - "3000:3000"  # FalkorDB web UI
    volumes:
      - falkordb_data:/data
    environment:
      - FALKORDB_PASSWORD=${FALKORDB_PASSWORD:-}
    healthcheck:
      test: ["CMD", "redis-cli", "-p", "6379", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
    networks:
      - graphiti-network

  graphiti-mcp:
    image: zepai/knowledge-graph-mcp:standalone
    container_name: graphiti-mcp
    env_file:
      - path: .env
        required: false
    depends_on:
      falkordb:
        condition: service_healthy
    environment:
      - FALKORDB_URI=${FALKORDB_URI:-redis://falkordb:6379}
      - FALKORDB_PASSWORD=${FALKORDB_PASSWORD:-}
      - FALKORDB_DATABASE=${FALKORDB_DATABASE:-default_db}
      - GRAPHITI_GROUP_ID=${GRAPHITI_GROUP_ID:-main}
      - SEMAPHORE_LIMIT=${SEMAPHORE_LIMIT:-10}
      - CONFIG_PATH=/app/mcp/config/config.yaml
      - PATH=/root/.local/bin:${PATH}
    ports:
      - "8000:8000"  # MCP server HTTP transport
    command: ["uv", "run", "main.py"]
    networks:
      - graphiti-network

volumes:
  falkordb_data:
    driver: local

networks:
  graphiti-network:
    driver: bridge
```

### Step 3: Configure Environment Variables

Create `~/ai-hub/graphiti/.env` in the same directory as docker-compose.yml:

```bash
# Required for embeddings (OpenAI API key)
OPENAI_API_KEY=sk-your-openai-api-key-here

# Neo4j Configuration (if using Neo4j backend)
NEO4J_URI=bolt://neo4j:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=demodemo
NEO4J_DATABASE=neo4j
NEO4J_PORT=7687

# FalkorDB Configuration (if using FalkorDB backend)
FALKORDB_URI=redis://falkordb:6379
FALKORDB_PASSWORD=
FALKORDB_DATABASE=default_db

# Graphiti Application Settings
GRAPHITI_GROUP_ID=main
SEMAPHORE_LIMIT=10
USE_PARALLEL_RUNTIME=true
MAX_REFLEXION_ITERATIONS=3

# Optional: Additional API keys for extended features
ANTHROPIC_API_KEY=
```

**Important:** Based on the production-tested ailocalstack configuration, the MCP server reads these environment variables directly from the `.env` file via Docker Compose's `env_file` directive. The running configuration uses `zepai/knowledge-graph-mcp:standalone` on port 8000.

**Port Reference:**
- MCP Server: `8000` (HTTP SSE endpoint)
- Neo4j HTTP: `7474` (Browser interface)
- Neo4j Bolt: `7687` (Database protocol)
- FalkorDB Redis: `6379` (Database protocol)
- FalkorDB Web UI: `3000` (Browser interface)

Verify configuration file exists:

```bash
ls -la ~/ai-hub/graphiti/.env
# Should show the .env file

cat ~/ai-hub/graphiti/.env | grep -E "^(OPENAI_API_KEY|NEO4J_|FALKORDB_)" | cut -d= -f1
# Should list all configured variables
```

### Step 4: Start MCP Server

```bash
cd ~/ai-hub/graphiti

# Start services
docker-compose up -d

# Wait for initialization (30-60 seconds)
echo "Waiting for services to be healthy..."
sleep 30

# Check status
docker-compose ps

# View logs
docker-compose logs -f graphiti-mcp
```

**Expected Output:**
```
NAME                COMMAND                  SERVICE             STATUS              PORTS
graphiti-mcp        "uv run main.py"         graphiti-mcp        running (healthy)   0.0.0.0:8000->8000/tcp
graphiti-neo4j      "/startup/docker-entr…"  neo4j               running (healthy)   0.0.0.0:7474->7474/tcp, 0.0.0.0:7687->7687/tcp
```

### Step 5: Verify MCP Server

```bash
# Health check
curl http://127.0.0.1:8000/health

# Expected: {"status": "healthy"}

# List available tools
curl http://127.0.0.1:8000/tools 2>/dev/null | head -20
```

## OpenCode Configuration

### Step 1: Configure MCP Client

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

**Note:** If using FalkorDB on port 8001, update the URL:

```json
"command": [
  "npx",
  "-y",
  "mcp-remote",
  "http://127.0.0.1:8001/sse",
  "--allow-http"
]
```

### Step 2: Validate Configuration

```bash
# Check JSON syntax
npx jsonlint ~/.config/opencode/opencode.json

# Or use Python
python3 -m json.tool ~/.config/opencode/opencode.json > /dev/null && echo "Valid JSON"
```

### Step 3: Configure Agent Prompts (Optional)

For oh-my-opencode users, edit `~/.config/opencode/oh-my-opencode.json`:

```json
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json",
  "agents": {
    "sisyphus": {
      "model": "kimi/kimi-k2.5-coding",
      "prompt": "Load Node Sage memory context first via skill(\"graphiti-memory\"). Use the name \"Node Sage\" in user-facing text."
    },
    "explore": {
      "model": "kimi/kimi-k2.5-coding",
      "prompt": "Load Node Sage memory context first via skill(\"graphiti-memory\"). Use the name \"Node Sage\" in user-facing text."
    },
    "atlas": {
      "model": "kimi/kimi-k2.5-coding",
      "prompt": "Load Node Sage memory context first via skill(\"graphiti-memory\"). Use the name \"Node Sage\" in user-facing text."
    }
  }
}
```

## Integration with auto-worklog

Node Sage works as an **optional companion** to auto-worklog. Here's how to integrate them:

### AGENTS.md Configuration

Add to your `~/.config/opencode/AGENTS.md`:

```markdown
## Startup Behavior

### Session Initialization

On every user interaction, the following agents/skills are automatically invoked:

1. **worklogger** (REQUIRED) - Initializes worklog for current session
   - Detects work intent from user message
   - Creates worklog entry with timestamp and context
   - Source: auto-worklog skill

2. **Node Sage** (OPTIONAL) - Loads persistent memory context
   - Retrieves relevant context from previous sessions
   - Available when MCP server is running
   - Falls back gracefully if unavailable
   - Source: graphiti-memory skill

### Optional Enhancement

When Node Sage is enabled alongside auto-worklog:
- Worklog entries include richer context
- Cross-session continuity for long-running tasks
- Team knowledge sharing via graph database

### Configuration

```bash
# Required for auto-worklog
export AUTO_WORKLOG_DIR="$HOME/.local/share/ai-hub/worklog/"

# Required for Node Sage (MCP server)
export OPENAI_API_KEY="sk-..."
```

### Detection Priority

1. **worklogger** - ALWAYS active, creates worklog on work intent
2. **Node Sage** - Active when available, enriches context
```

### Usage Pattern

```typescript
// Session start - worklogger creates worklog
// Node Sage (if available) loads context

// Example workflow:
userMessage = "Continue working on the authentication feature"

// 1. worklogger auto-triggered
worklogPath = createWorklog({
  topic: "Authentication feature",
  request: userMessage
})

// 2. Node Sage loads previous context (optional)
if (mcpServerAvailable) {
  await skill("graphiti-memory")
  context = search_memory_facts(
    query="authentication feature previous work",
    group_ids=["project:myapp"]
  )
}

// 3. Proceed with work using available context
```

## Verification

### Test 1: MCP Server Health

```bash
# Health endpoint
curl http://127.0.0.1:8000/health
echo

# Expected: {"status": "healthy"}
```

### Test 2: Skill Loading

Start OpenCode and verify skill loads:

```
User: Initialize Node Sage
```

**Expected:** Agent responds with Node Sage context loaded.

### Test 3: Memory Operations

```
User: Store this in memory: We are using PostgreSQL 15 for this project
```

Then:
```
User: What database are we using?
```

**Expected:** Agent retrieves "PostgreSQL 15" from memory.

### Test 4: Worklog + Node Sage Integration

```
User: Work on implementing user authentication
```

**Expected:**
1. Worklog created automatically
2. Node Sage loads any previous auth-related context (if available)
3. Agent proceeds with full context

## Troubleshooting

### Issue: MCP Server Won't Start

**Symptom:** `docker-compose up` fails or containers exit immediately

**Solution:**
```bash
# Check logs
docker-compose logs

# Common issues:
# 1. Port already in use
lsof -i :8000  # Kill process if needed

# 2. Missing environment variables
echo $OPENAI_API_KEY

# 3. Permission issues
docker-compose down -v
docker-compose up -d
```

### Issue: OpenCode Can't Connect to MCP

**Symptom:** "MCP server not available" or connection errors

**Solution:**
```bash
# 1. Verify MCP server is running
curl http://127.0.0.1:8000/health

# 2. Check opencode.json syntax
python3 -m json.tool ~/.config/opencode/opencode.json

# 3. Verify mcp-remote is installed
npx mcp-remote --version

# 4. Restart OpenCode
```

### Issue: Memory Not Persisting

**Symptom:** Saved memory not found in subsequent sessions

**Solution:**
```bash
# 1. Check database persistence
docker volume ls | grep graphiti

# 2. Verify group IDs are consistent
# Check worklog for exact group_id used

# 3. Check database directly
docker exec -it graphiti-neo4j-1 cypher-shell -u neo4j -p $NEO4J_PASSWORD
# Run: MATCH (n) RETURN count(n);
```

### Issue: High Memory Usage

**Symptom:** System slow, out of memory errors

**Solution:**
```bash
# Check resource usage
docker stats --no-stream

# Limit Neo4j memory (edit docker-compose.yml)
environment:
  - NEO4J_dbms_memory_heap_initial__size=1G
  - NEO4J_dbms_memory_heap_max__size=2G

# Or switch to FalkorDB (lighter weight)
```

### Issue: Port Conflicts

**Symptom:** "Bind for 0.0.0.0:8000 failed" or similar

**Solution:**
```bash
# Find process using port
lsof -i :8000

# Kill process or change port in docker-compose.yml
# Change: ports:
#   - "8001:8000"  # Host:Container
```

## Maintenance

### Backup Memory

```bash
# Create backup directory
mkdir -p ~/ai-hub/backups

# Neo4j backup
docker exec graphiti-neo4j neo4j-admin dump --to=/tmp/backup.dump
docker cp graphiti-neo4j:/tmp/backup.dump ~/ai-hub/backups/neo4j-$(date +%Y%m%d).dump

# Or backup entire volume
docker run --rm \
  -v graphiti_neo4j_data:/data \
  -v ~/ai-hub/backups:/backups \
  alpine tar czf /backups/neo4j-$(date +%Y%m%d).tar.gz -C /data .
```

### Update MCP Server

```bash
cd ~/ai-hub/graphiti

# Pull latest images
docker-compose pull

# Restart with new images
docker-compose up -d

# Verify
docker-compose ps
```

### Reset All Data

```bash
# WARNING: Destructive - deletes all memory!
cd ~/ai-hub/graphiti
docker-compose down -v  # -v removes volumes
docker-compose up -d
```

## Next Steps

1. **Explore Graph Database:** Visit http://localhost:7474 (Neo4j browser)
2. **Read Full Documentation:** See [SKILL.md](../.agents/skills/graphiti-memory/SKILL.md)
3. **Integration Examples:** Check auto-worklog integration patterns
4. **Backup Strategy:** Set up automated backups

---

**Integration Guide Version:** 1.0  
**Last Updated:** 2025-03-15  
**Repository:** https://github.com/GavinTomlins/ai-hub
