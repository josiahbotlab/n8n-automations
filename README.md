# n8n Automations

n8n workflow development repo using Claude Code with build-test-fix loop.

## Setup

```bash
export N8N_API_KEY="your-n8n-api-key"
```

## Usage

Build a workflow from natural language:

```
/build-n8n-workflow Send a Telegram message every morning at 9am with the weather forecast
```

Claude will plan the nodes, build the JSON, push to n8n, test-execute, fix any errors, and save the working workflow to `workflows/`.

## Structure

```
workflows/     # Completed workflow JSONs
.claude/       # Claude Code config and slash commands
CLAUDE.md      # Project instructions
```

## n8n Instance

https://n8n-render-esib.onrender.com
