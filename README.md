# Serena MCP Server Setup

Serena is an MCP (Model Context Protocol) server that provides code-aware tools to Claude Code, including symbol navigation, refactoring, and language server features.

This project runs Serena in Docker and makes it available to all projects under your dev directory.

## Prerequisites

- Docker and Docker Compose installed
- Claude Code CLI installed

## Project Structure

```
claude-mcp/
├── project                     # CLI script for managing Serena
├── docker-compose.yml          # Docker service definition
├── config/
│   └── serena_config.yml       # Serena configuration (auto-managed)
└── README.md
```

## Step 1: Start Serena

```bash
./project start
```

Verify it's running:

```bash
./project logs
```

You should see:

```
Uvicorn running on http://0.0.0.0:9121 (Press CTRL+C to quit)
```

Open the web dashboard:

```bash
./project dashboard
```

Or visit http://localhost:24282/dashboard/ directly.

## Step 2: Add Serena to Claude Code (Global)

This makes Serena available in every Claude Code session.

Create or edit `~/.claude/.mcp.json`:

```json
{
  "mcpServers": {
    "serena": {
      "type": "sse",
      "url": "http://localhost:9121/sse"
    }
  }
}
```

Restart any running Claude Code sessions for the change to take effect.

## Step 3: Activate a Project

When you start a Claude Code session in any project, tell Claude:

```
activate project /workspaces/projects/<project-folder-name>
```

For example:

```
activate project /workspaces/projects/lead-distribution-core
```

The path must use `/workspaces/projects/` because that's where your dev directory is mounted inside the container.

Once activated, Serena registers the project and it persists across restarts.

## Adding New Projects

Any project in projects directory is automatically available — no config changes needed. Just activate it by name in your Claude Code session.

```yaml
volumes:
  - ./config:/workspaces/serena/config
  - ../:/workspaces/projects
```

Then restart: `./project restart`

## CLI Reference

All commands are run from the `claude-mcp` directory.

| Action         | Command               |
| -------------- | --------------------- |
| Start          | `./project start`     |
| Stop           | `./project stop`      |
| Restart        | `./project restart`   |
| View logs      | `./project logs`      |
| Check status   | `./project status`    |
| Open dashboard | `./project dashboard` |
| Open shell     | `./project shell`     |
| Pull update    | `./project pull`      |
| Remove         | `./project down`      |
| Help           | `./project help`      |

## Troubleshooting

### MCP shows "failed" in Claude Code

- Verify Serena is running: `docker ps --filter name=serena-mcp`
- Test the SSE endpoint: `curl -s http://localhost:9121/sse`
- Make sure the config uses `"type": "sse"` and URL ends with `/sse`

### Dashboard not loading

- Check that port 24282 isn't used by another process: `lsof -i :24282`
- Do NOT use `network_mode: host` on macOS — it doesn't forward ports to the host

### Project not showing in dashboard

- Make sure you used the container path (`/workspaces/projects/...`), not your Mac path
- Check that the project directory exists inside the container: `docker exec serena-mcp ls /workspaces/projects/`

### Config errors on startup

Check logs for details: `docker logs serena-mcp`. The `serena_config.yml` must have a `projects` key (can be empty: `projects: []`).
