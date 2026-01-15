# DORA Metrics Collection & Analysis

This repository collects **DORA metrics** (Deployment Frequency, Lead Time for Changes, Change Failure Rate, MTTR) from CI/CD pipelines to measure team DevOps performance.

## 📊 What is DORA?

DORA (DevOps Research and Assessment) metrics are four key indicators of software delivery performance:

| Metric | Description | Ideal Range |
|--------|-------------|-------------|
| **Deployment Frequency** | How often code is deployed to production | On-demand / Multiple per day |
| **Lead Time for Changes** | Time from commit to deployment | < 1 hour |
| **Change Failure Rate** | % of deployments that cause failures | < 15% |
| **MTTR** (Mean Time to Recovery) | Average time to recover from failure | < 1 hour |

## 🔄 How It Works

1. **Backend & Frontend CI/CD pipelines** run tests and deployments
2. **On success/failure**, they send a DORA event via `repository_dispatch` to this repo
3. **This repo's `collect-dora` workflow** receives the event and appends it to `dora_metrics.csv`
4. **A report is generated** summarizing all metrics

## 📥 Event Format

When CI/CD pipelines send DORA events, they include:

```json
{
  "event_type": "dora-event",
  "client_payload": {
    "service": "backend" | "frontend",
    "event": "deploy",
    "status": "success" | "fail",
    "commit": "abc123...",
    "timestamp": "2026-01-15T12:00:00+00:00"
  }
}
```

Example CSV row:
```
backend,deploy,success,abc123def456,2026-01-15T12:00:00+00:00
```

## 📋 CSV Columns

| Column | Example | Notes |
|--------|---------|-------|
| `service` | `backend`, `frontend` | Which service deployed |
| `event` | `deploy` | Event type (currently only deploy) |
| `status` | `success`, `fail` | Deployment result |
| `commit` | `abc123def456` | Git commit SHA (first 7 chars) |
| `timestamp` | `2026-01-15T12:00:00Z` | When event occurred (ISO 8601) |

## 🚀 Setup

### 1. Add Token to Backend & Frontend Repos

Each CI/CD pipeline needs a token to send events to this repo:

1. **Create a Personal Access Token (PAT)**:
   - GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
   - Scopes: `repo` (or `public_repo` if repos are public)
   - Copy the token

2. **Add to each repo's secrets**:
   - **Backend repo** (`iRent-Backend`):
     - Settings → Secrets and variables → Actions
     - New secret: `DORA_METRICS_TOKEN` = your PAT
     - New secret: `DORA_METRICS_REPO` = `Tarik-3/dora-metrics`
   
   - **Frontend repo** (`iRentFrontend`):
     - Same secrets as above

### 2. Verify CI/CD Workflows

Check that backend/frontend workflows have these steps:

**Backend** (`.github/workflows/backend-ci-cd.yml`):
```yaml
- name: Send DORA event to dora-metrics (success)
  if: success()
  env:
    DORA_TOKEN: ${{ secrets.DORA_METRICS_TOKEN }}
  run: |
    # Sends event to dora-metrics repo
    curl -s -X POST \
      -H "Authorization: token $DORA_TOKEN" \
      https://api.github.com/repos/${{ env.DORA_METRICS_REPO }}/dispatches \
      -d '{"event_type":"dora-event", ...}'
```

**Frontend** (`.github/workflows/frontend-ci-cd.yml`):
- Same pattern as backend

### 3. Verify This Repo's Workflow

Check `.github/workflows/collect-dora.yml`:
```yaml
on:
  repository_dispatch:
    types: [dora-event]
```

This listens for incoming events.

## 📈 Viewing Metrics

After deployments happen:

1. **Check the Actions tab**: `https://github.com/Tarik-3/dora-metrics/actions`
2. **Look for `collect-dora` runs**: Each deployment triggers a run
3. **Download the report**: 
   - Click a completed run → Artifacts → `dora-report`
   - Or open `DORA_REPORT.md` in this repo

## 📊 Analyzing Metrics

### Deployment Frequency
- **High frequency** (multiple per day) = Good (continuous delivery)
- **Low frequency** (< weekly) = May need improvement

### Lead Time
- **< 1 hour** = Elite performer
- **1-24 hours** = High performer
- **> 24 hours** = Needs improvement

### Change Failure Rate
- **< 15%** = Good
- **15-45%** = Acceptable
- **> 45%** = Needs improvement

### MTTR
- **< 1 hour** = Elite
- **1-24 hours** = Acceptable
- **> 24 hours** = Needs improvement

## 🔍 Troubleshooting

### Events Not Being Recorded?

1. **Check `DORA_METRICS_TOKEN`** in backend/frontend repo secrets:
   - Must be a valid Personal Access Token
   - Must have `repo` scope
   - Must be able to trigger events in `dora-metrics` repo

2. **Check `DORA_METRICS_REPO`** value:
   - Should be exactly: `Tarik-3/dora-metrics`

3. **Check CI/CD logs** for the `Send DORA event` step:
   - Backend: `.github/workflows/backend-ci-cd.yml`
   - Frontend: `.github/workflows/frontend-ci-cd.yml`

### Report Not Generating?

1. **Check `collect-dora` workflow**: Actions tab in this repo
2. **Look for errors** in the run logs
3. **Ensure `dora_metrics.csv` exists**: Should have at least a header row

## 🛠️ Manual Event Submission

To test or manually record an event:

```bash
curl -X POST \
  -H "Authorization: token YOUR_PAT" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/Tarik-3/dora-metrics/dispatches \
  -d '{
    "event_type": "dora-event",
    "client_payload": {
      "service": "backend",
      "event": "deploy",
      "status": "success",
      "commit": "abc123",
      "timestamp": "2026-01-15T12:00:00Z"
    }
  }'
```

## 📚 References

- [DORA Metrics](https://cloud.google.com/architecture/devops-measurement-capabilities-in-cloud)
- [GitHub Repository Dispatch](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#repository_dispatch)
- [GitHub Actions Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

## 📝 Next Steps

1. ✅ Add `DORA_METRICS_TOKEN` to backend/frontend repo secrets
2. ✅ Add `DORA_METRICS_REPO` to backend/frontend repo secrets  
3. ✅ Trigger a deployment (push to `feature/pipeline`)
4. ✅ Check Actions tab in this repo for `collect-dora` run
5. ✅ Review `DORA_REPORT.md` artifacts

---

**Questions?** Check the CI/CD workflow logs or the DORA metrics guide.
