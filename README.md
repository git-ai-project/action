# Git AI Dashboards

Track AI-assisted development metrics and export OpenTelemetry data for your GitHub repositories.

## Quick Start

Copy this workflow file to `.github/workflows/git-ai.yml` in your repository:

```yaml
name: Git AI Dashboards
on:
  pull_request:
    types: [closed]
  schedule:
    - cron: "0 0 * * *" # Daily at midnight UTC
  workflow_dispatch:

jobs:
  pr-close:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    continue-on-error: true
    permissions:
      contents: write
    steps:
      - name: Run Git AI PR Close
        uses: git-ai-project/action/pr-close@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          pr-url: ${{ github.event.pull_request.html_url }}
          merge-commit-sha: ${{ github.event.pull_request.merge_commit_sha }}
          OTEL_EXPORTER_OTLP_ENDPOINT: ${{ secrets.OTEL_EXPORTER_OTLP_ENDPOINT }}
          OTEL_EXPORTER_OTLP_HEADERS: ${{ secrets.OTEL_EXPORTER_OTLP_HEADERS }}
          OTEL_SERVICE_NAME: "git-ai-dashboards"

  daily-metrics:
    if: github.event_name == 'schedule' || github.event_name == 'workflow_dispatch'
    runs-on: ubuntu-latest
    continue-on-error: true
    permissions:
      contents: read
    steps:
      - name: Run Git AI Daily Metrics
        uses: git-ai-project/action/daily-metrics@v1
        with:
          OTEL_EXPORTER_OTLP_ENDPOINT: ${{ secrets.OTEL_EXPORTER_OTLP_ENDPOINT }}
          OTEL_EXPORTER_OTLP_HEADERS: ${{ secrets.OTEL_EXPORTER_OTLP_HEADERS }}
          OTEL_SERVICE_NAME: "git-ai-dashboards"

```

## Actions

This repository provides two specialized actions:

### `git-ai-project/action/pr-close`

Runs when a pull request is merged. Analyzes the PR for AI-assisted contributions and exports metrics.

#### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `github-token` | No | `${{ github.token }}` | GitHub token for API access |
| `pr-url` | **Yes** | - | The pull request URL (use `${{ github.event.pull_request.html_url }}`) |
| `merge-commit-sha` | **Yes** | - | The merge commit SHA (use `${{ github.event.pull_request.merge_commit_sha }}`) |
| `repo-url` | No | Current repository | Repository URL |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | No | - | OpenTelemetry endpoint URL |
| `OTEL_EXPORTER_OTLP_HEADERS` | No | - | OpenTelemetry headers (key=value format, comma separated) |
| `OTEL_SERVICE_NAME` | No | - | OpenTelemetry service name |

#### Required Permissions

```yaml
permissions:
  contents: write  # Required to read repository and commit history
```

---

### `git-ai-project/action/daily-metrics`

Runs on a schedule to export aggregate daily metrics for the repository.

#### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `repo-url` | No | Current repository | Repository URL |
| `default-branch` | No | Repository default branch | Default branch name to analyze |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | No | - | OpenTelemetry endpoint URL |
| `OTEL_EXPORTER_OTLP_HEADERS` | No | - | OpenTelemetry headers (key=value format, comma separated) |
| `OTEL_SERVICE_NAME` | No | - | OpenTelemetry service name |

#### Required Permissions

```yaml
permissions:
  contents: read  # Required to read repository and commit history
```

## OpenTelemetry Configuration

To export metrics to your observability platform, add the following secrets to your repository:

1. Go to **Settings** → **Secrets and variables** → **Actions**
2. Add the following secrets:

| Secret | Description |
|--------|-------------|
| `OTEL_EXPORTER_OTLP_ENDPOINT` | Your OTLP endpoint URL |
| `OTEL_EXPORTER_OTLP_HEADERS` | Authentication headers |

### Provider Examples

#### Grafana Cloud

```
OTEL_EXPORTER_OTLP_ENDPOINT: https://otlp-gateway-prod-us-central-0.grafana.net/otlp
OTEL_EXPORTER_OTLP_HEADERS: Authorization=Basic <base64-encoded-credentials>
```

#### Honeycomb

```
OTEL_EXPORTER_OTLP_ENDPOINT: https://api.honeycomb.io
OTEL_EXPORTER_OTLP_HEADERS: x-honeycomb-team=<your-api-key>
```

#### Datadog

```
OTEL_EXPORTER_OTLP_ENDPOINT: https://otel.datadoghq.com
OTEL_EXPORTER_OTLP_HEADERS: DD-API-KEY=<your-api-key>
```

#### New Relic

```
OTEL_EXPORTER_OTLP_ENDPOINT: https://otlp.nr-data.net
OTEL_EXPORTER_OTLP_HEADERS: api-key=<your-license-key>
```
