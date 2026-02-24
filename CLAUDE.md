# n8n Automations

## n8n Instance
- **URL**: https://n8n-render-esib.onrender.com
- **Auth**: `X-N8N-API-KEY` header, key from env var `N8N_API_KEY`
- **API base**: `/api/v1`

## API Endpoints

```bash
# Workflows
GET    /api/v1/workflows                    # List all
GET    /api/v1/workflows/{id}               # Get one
POST   /api/v1/workflows                    # Create (always set "active": false)
PATCH  /api/v1/workflows/{id}               # Update
PATCH  /api/v1/workflows/{id}  {"active": true/false}  # Activate/deactivate
POST   /api/v1/workflows/{id}/run           # Execute
DELETE /api/v1/workflows/{id}               # Delete

# Executions
GET    /api/v1/executions?workflowId={id}&limit=10  # List
GET    /api/v1/executions/{executionId}              # Get details
```

All requests include: `-H "X-N8N-API-KEY: $N8N_API_KEY"`

## Build-Test-Fix Loop

When building any workflow, follow this cycle:

1. **Plan** — show me the node layout and logic before writing JSON
2. **Build** — construct the workflow JSON
3. **Show** — display the full JSON and explain each node before pushing
4. **Create** — POST to `/api/v1/workflows` with `"active": false`
5. **Execute** — POST to `/api/v1/workflows/{id}/run` to test
6. **Read results** — GET `/api/v1/executions/{executionId}` and check for errors
7. **Fix** — if execution failed, patch the workflow and re-execute
8. **Repeat** steps 5-7 until the execution succeeds
9. **Save** — write the final working JSON to `workflows/` with a descriptive filename

## Rules

- Always show me the plan before building
- Always show the workflow JSON before pushing to n8n
- Store completed workflow JSONs in `workflows/` as `{descriptive-name}.json`
- Use `jq` for parsing all API responses
- Use `--connect-timeout 15 --max-time 60` on all curl calls
- Render may cold start (30-60s) — retry on timeout or 5xx up to 3 times with 5s sleep between attempts
- Never activate a workflow without my explicit approval
