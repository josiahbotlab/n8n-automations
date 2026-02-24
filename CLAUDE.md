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

## Credentials

These are n8n credential IDs from the n8n UI (Settings → Credentials). Replace the placeholder values after creating them in n8n.

```
TELEGRAM_CREDENTIAL_ID=TODO        # Credentials → Telegram API → copy ID from URL
TELEGRAM_CREDENTIAL_NAME=TODO      # Display name you gave it (e.g. "Telegram Bot")
SMTP_CREDENTIAL_ID=TODO            # Credentials → SMTP → copy ID from URL
SMTP_CREDENTIAL_NAME=TODO          # Display name (e.g. "Gmail SMTP")
GOOGLE_SHEETS_CREDENTIAL_ID=TODO   # Credentials → Google Sheets OAuth2 → copy ID
GOOGLE_SHEETS_CREDENTIAL_NAME=TODO
HTTP_HEADER_AUTH_CREDENTIAL_ID=TODO  # For authenticated HTTP requests
HTTP_HEADER_AUTH_CREDENTIAL_NAME=TODO
```

When building workflow JSON that uses credentials, reference them as:
```json
"credentials": {
  "telegramApi": {
    "id": "TELEGRAM_CREDENTIAL_ID",
    "name": "TELEGRAM_CREDENTIAL_NAME"
  }
}
```

## Test Data

Replace these placeholders before testing workflows that send messages or emails.

```
MY_EMAIL=TODO                      # For email-sending workflows
MY_TELEGRAM_CHAT_ID=TODO           # Numeric chat ID (DM @userinfobot to get yours)
WEBHOOK_TEST_PAYLOAD='{"event": "test", "data": {"message": "hello", "timestamp": "2024-01-01T00:00:00Z"}}'
WEBHOOK_FORM_PAYLOAD='name=Test+User&email=test@example.com&message=Hello+World'
```

## n8n Gotchas

Things that will silently break workflows if you get them wrong:

**Expressions**: n8n expressions use `={{ }}` syntax, NOT `${ }`. Always prefix with `=`:
- Correct: `={{ $json.name }}`
- Wrong: `{{ $json.name }}` (treated as literal string)
- Wrong: `=${ $json.name }` (JS template literal, not n8n expression)

**Node names in connections**: Connection targets must match node names EXACTLY (case-sensitive, including spaces). If you name a node `"Send Email"`, the connection must reference `"Send Email"`, not `"send email"` or `"Send_Email"`.

**Credential IDs**: Credentials in workflow JSON need both `id` (string) and `name` (string). If either is wrong or missing, the node silently uses no auth. Always use the exact values from the n8n UI.

**Code node syntax**: The Code node (`n8n-nodes-base.code` v2) uses:
- `$input.all()` to get all input items
- `$input.first()` for the first item
- Must return an array of objects with `json` key: `return [{json: {key: "value"}}]`
- Do NOT use `items` variable (that's v1 syntax)
- `$env.VAR_NAME` to read n8n environment variables

**Webhook paths must be unique**: Two active workflows cannot share the same webhook path. Use descriptive paths like `/api/my-workflow-name` not generic ones like `/webhook`.

**Render cold starts**: The n8n instance on Render spins down after inactivity. First request after idle can take 30-60 seconds. Always use `--max-time 90` for the first health check or workflow execution after a gap. Subsequent requests are fast.

**Empty execution data**: When a workflow executes via API (`/run`), the response returns an execution ID but NOT the results. You must separately GET `/api/v1/executions/{id}` to read the output. Poll if `finished` is false.

**PATCH replaces nodes array**: When updating a workflow via PATCH, the `nodes` and `connections` arrays are replaced entirely, not merged. Always send the complete workflow, not partial updates.

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
