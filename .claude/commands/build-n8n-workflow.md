Build an n8n workflow from the following description: $ARGUMENTS

Follow this process exactly:

1. **Plan**: Break down the description into n8n nodes. Identify the trigger type, processing steps, and output actions. Show me the plan with node names, types, and how they connect. Wait for my approval before proceeding.

2. **Build**: Construct the full workflow JSON. Every workflow must have:
   - `"active": false`
   - `"settings": {"executionOrder": "v1"}`
   - Nodes positioned on a grid (start at [250,300], space 250px horizontally)
   - All connections properly wired

3. **Show**: Display the complete JSON and explain what each node does. Wait for my approval.

4. **Create**: Push to n8n via API:
   ```bash
   curl -s --connect-timeout 15 --max-time 60 \
     -X POST -H "X-N8N-API-KEY: $N8N_API_KEY" -H "Content-Type: application/json" \
     -d @workflow.json \
     "https://n8n-render-esib.onrender.com/api/v1/workflows" | jq '{id, name, active}'
   ```
   Save the returned workflow ID.

5. **Execute**: Test the workflow:
   ```bash
   curl -s --connect-timeout 15 --max-time 60 \
     -X POST -H "X-N8N-API-KEY: $N8N_API_KEY" -H "Content-Type: application/json" \
     -d '{"data": {}}' \
     "https://n8n-render-esib.onrender.com/api/v1/workflows/{id}/run" | jq '{executionId: .id}'
   ```

6. **Check results**: Read the execution output:
   ```bash
   curl -s --connect-timeout 15 --max-time 60 \
     -H "X-N8N-API-KEY: $N8N_API_KEY" \
     "https://n8n-render-esib.onrender.com/api/v1/executions/{executionId}" \
     | jq '{status: .status, finished: .finished, data: .data}'
   ```

7. **Fix loop**: If the execution failed:
   - Read the error from the execution data
   - Fix the workflow JSON
   - PATCH the workflow via API
   - Re-execute and re-check
   - Repeat until the execution succeeds

8. **Save**: Write the final working workflow JSON to `workflows/{descriptive-name}.json` and report success with the workflow ID and execution results.

Retry any curl call up to 3 times on timeout or 5xx errors (Render cold starts can take 30-60s).
