# Deployment Information

## Public URL
https://ai-agent-nilc.onrender.com

## Platform
Render

## Service Info
- Service type: Web Service (Blueprint)
- Config source: 03-cloud-deployment/render/render.yaml

## Test Commands

### Health Check
```bash
curl https://ai-agent-nilc.onrender.com/health
# Expected: {"status":"ok", ...}
```

Sample response:
```json
{"status":"ok","uptime_seconds":361.5,"platform":"Railway","timestamp":"2026-04-17T13:02:05.408781+00:00"}
```

### API Test
```bash
curl -X POST https://ai-agent-nilc.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello from Render"}'
```

Sample response:
```json
{"question":"Hello from Render","answer":"Agent ... (mock response)","platform":"Railway"}
```

## Environment Variables Set
- ENVIRONMENT=production
- PYTHON_VERSION=3.11.0
- AGENT_API_KEY=(Render generate value)
- OPENAI_API_KEY=(optional for this mock lab)

## Actual Validation Results
- `GET /health` on public URL: `200 OK`
- `POST /ask` with JSON body: `200 OK`
- Public service currently follows Part 3 cloud app contract (`question` in JSON body).

PowerShell command used successfully:
```powershell
$body = @{ question = "Hello from Render" } | ConvertTo-Json
Invoke-RestMethod -Uri "https://ai-agent-nilc.onrender.com/ask" -Method Post -ContentType "application/json" -Body $body
```

Note:
- API authentication/rate-limit/cost-guard verification was executed in local production modules (Part 4/5) and documented in `MISSION_ANSWERS.md`.

## Render Setup Steps (Done)
1. Push repository to GitHub
2. Open Render Dashboard
3. New -> Blueprint
4. Connect repository
5. Render reads render.yaml
6. Set environment variables
7. Deploy and wait for Live status

## Evidence / Screenshots
- Deployment dashboard: screenshots/render-dashboard.png
- Service live status: screenshots/render-live.png
- Health check test: screenshots/render-health-test.png
- Ask endpoint test: screenshots/render-ask-test.png

## Notes
- Local verification before cloud deploy was completed:
  - GET /health returns status ok
  - POST /ask works with JSON body
- If service is sleeping (free tier), first request may be slow.
