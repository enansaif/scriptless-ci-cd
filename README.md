# Scriptless CI/CD (test harness)

Minimal GitHub repo to trigger [Scriptless](https://dev-scriptless.nymphsolutions.com) **release runs** from GitHub Actions using a user CI token (`nlu_…`).

## One-time setup

### 1. Create a CI token in Scriptless

1. Open **Integrations** in dev Scriptless.
2. Create an integration token and copy the full `nlu_…` value (shown once).

### 2. Add repository secrets

In this repo: **Settings → Secrets and variables → Actions → New repository secret**

| Secret | Dev example value |
|--------|-------------------|
| `NLU_API_TOKEN` | Your `nlu_…` token |
| `NLU_API_URL` | `https://dev-api-scriptless.nymphsolutions.com` |
| `NLU_RELEASE_ID` | UUID from the release page URL (`/releases/{id}`) |

Example release (dev): `a4cb94f8-2416-42a0-a1f5-4ad2b0039828`

The token owner must have access to that release’s organization.

### 3. Run the workflow

**Actions → Run NLU release (dev) → Run workflow**

Or push to `main` (workflow also runs on push).

## What it does

1. `POST /api/ci/releases/{id}/run` on the backend.
2. Polls `GET /api/ci/runs/{run_id}` until the run finishes.
3. Fails the job if conclusion is `failure` or `cancelled`.

When you add or remove test cases from the release, Scriptless bumps the release version; you do **not** need to change `NLU_RELEASE_ID`.

## Local smoke test (optional)

```bash
export NLU_API_URL="https://dev-api-scriptless.nymphsolutions.com"
export NLU_API_TOKEN="nlu_…"
export NLU_RELEASE_ID="your-release-uuid"

curl -sS -X POST "${NLU_API_URL%/}/api/ci/releases/${NLU_RELEASE_ID}/run" \
  -H "Authorization: Bearer ${NLU_API_TOKEN}" \
  -H "Content-Type: application/json" | jq .
```

## Files

- `github-action/` — composite action (curl + poll).
- `.github/workflows/nlu-release.yml` — dev workflow.

Upstream copy lives in [AttenSysAI/nlu-test-agent](https://github.com/AttenSysAI/nlu-test-agent) (`github-action/`).
