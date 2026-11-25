# YQ Web Studio — Org-Wide Engineering Standards

Central reusable GitHub Actions workflows shared across all **YQ-Web-Studio** and **Muslim-Atlas** repositories.

## Reusable PR Review Stack (`pr-review.yml`)

Provides a \$0, 3-layer automated PR review pipeline:
1. **Deterministic Workflow & Security Audit:** `actionlint` + `zizmor` (medium+ severity).
2. **Adversarial PR-Agent Review:** Powered by `gemini/gemini-2.5-pro` (with `gemini/gemini-2.5-flash` fallback), strict edge-case & security rubric, whitespace-preserving pure-rebase guard, and automated review status sync.
3. **On-Demand CodeRabbit:** Repositories set `reviews.auto_review.enabled: false` in `.coderabbit.yaml` to preserve CodeRabbit's free quota for manual `@coderabbitai review` checks on high-risk PRs.

### Usage in Any Repository

Create `.github/workflows/pr-agent.yml`:

```yaml
name: PR-Agent

on:
  pull_request:
    types: [opened, reopened, synchronize, ready_for_review]
  issue_comment:
    types: [created]

permissions:
  contents: read
  issues: write
  pull-requests: write

concurrency:
  group: ${{ github.workflow }}-pr-${{ github.event.pull_request.number || github.event.issue.number || github.ref }}
  cancel-in-progress: true

jobs:
  review:
    uses: YQ-Web-Studio/.github/.github/workflows/pr-review.yml@main
    secrets: inherit
```

### Required Secrets
- `GEMINI_API_KEY` (or `PR_AGENT_GEMINI_API_KEY`) configured at the organization or repository level.
