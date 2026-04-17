# Deployment Information

## Public URL
https://YOUR-RENDER-URL.onrender.com

## Platform
Render

## Service Info
- Service type: Web Service (Blueprint)
- Config source: 03-cloud-deployment/render/render.yaml

## Test Commands

### Health Check
```bash
curl https://YOUR-RENDER-URL.onrender.com/health
# Expected: {"status":"ok", ...}
```

### API Test
```bash
curl -X POST https://YOUR-RENDER-URL.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello from Render"}'
```

## Environment Variables Set
- PORT=8000
- AGENT_API_KEY=YOUR_SECRET_KEY
- LOG_LEVEL=INFO

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
