# Git AI Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Git%20AI-blue?logo=github)](https://github.com/marketplace/actions/git-ai)

Track AI-assisted development in your repository and export OpenTelemetry data for observability. This action helps you understand how AI tools are being used in your codebase by analyzing git commits and pull requests.

## Features

- 🔍 **AI Usage Tracking** - Automatically detects and tracks AI-assisted commits
- 📊 **OpenTelemetry Export** - Export metrics to any OTLP-compatible backend
- 🔄 **PR Integration** - Analyze merged pull requests for AI contribution data
- ⏰ **Scheduled Reports** - Daily exports of AI usage metrics

## Usage

### On Pull Request Merge

Add this workflow to track AI usage when pull requests are merged:

```yaml
name: Git AI

on:
  pull_request:
    types: [closed]

jobs:
  git-ai:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          ref: ${{ github.event.pull_request.merge_commit_sha }}

      - name: Run Git AI
        uses: acunniffe/git-ai-action@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          pr-url: ${{ github.event.pull_request.html_url }}
          merge-commit-sha: ${{ github.event.pull_request.merge_commit_sha }}
          # Optional: Configure OTLP endpoint
          # otel-endpoint: https://your-otlp-endpoint.com
          # otel-headers: "Authorization=Bearer token"
```

### Daily Scheduled Export

Run daily exports of AI usage metrics:

```yaml
name: Git AI Daily

on:
  schedule:
    - cron: "0 0 * * *" # Daily at midnight UTC
  workflow_dispatch: # Allow manual trigger

jobs:
  git-ai-daily:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run Git AI
        uses: acunniffe/git-ai-action@v1
        with:
          default-branch: ${{ github.event.repository.default_branch }}
          # Optional: Configure OTLP endpoint
          # otel-endpoint: https://your-otlp-endpoint.com
          # otel-headers: "Authorization=Bearer token"
```

### Complete Workflow (Both Modes)

A single workflow file that handles both PR merges and daily exports:

```yaml
name: Git AI

on:
  pull_request:
    types: [closed]
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch:

jobs:
  pr-tracking:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          ref: ${{ github.event.pull_request.merge_commit_sha }}

      - name: Run Git AI
        uses: acunniffe/git-ai-action@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          pr-url: ${{ github.event.pull_request.html_url }}
          merge-commit-sha: ${{ github.event.pull_request.merge_commit_sha }}

  daily-export:
    if: github.event_name == 'schedule' || github.event_name == 'workflow_dispatch'
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run Git AI
        uses: acunniffe/git-ai-action@v1
        with:
          default-branch: ${{ github.event.repository.default_branch }}
```

## Inputs

| Input               | Description                                         | Required | Default               |
| ------------------- | --------------------------------------------------- | -------- | --------------------- |
| `github-token`      | GitHub token for API access                         | ❌       | `${{ github.token }}` |
| `pr-url`            | Pull request URL (triggers PR mode when provided)   | ❌       | -                     |
| `repo-url`          | Repository URL                                      | ❌       | Auto-detected         |
| `merge-commit-sha`  | Merge commit SHA (required when pr-url is provided) | ❌       | -                     |
| `default-branch`    | Default branch name (for daily export mode)         | ❌       | `main`                |
| `otel-endpoint`     | OpenTelemetry OTLP endpoint URL                     | ❌       | -                     |
| `otel-headers`      | OpenTelemetry headers (comma-separated key=value)   | ❌       | -                     |
| `otel-service-name` | OpenTelemetry service name                          | ❌       | -                     |

## Mode Detection

The action automatically detects which mode to run:

- **PR Mode**: When `pr-url` is provided, the action runs PR tracking and export
- **Daily Mode**: When `pr-url` is not provided, the action runs daily export

## OpenTelemetry Configuration

To export metrics to your observability platform, configure the OTLP endpoint:

```yaml
- name: Run Git AI
  uses: acunniffe/git-ai-action@v1
  with:
    default-branch: main
    otel-endpoint: https://otlp.example.com:4317
    otel-headers: "Authorization=Bearer ${{ secrets.OTLP_TOKEN }}"
    otel-service-name: my-service
```

### Supported Backends

- Honeycomb
- Datadog
- New Relic
- Grafana Cloud
- Any OTLP-compatible backend

## How It Works

1. **Git Notes Analysis** - The action reads AI metadata stored in git notes (`refs/notes/ai/*`)
2. **Commit Analysis** - Analyzes commits for AI-assisted development patterns
3. **OTLP Export** - Exports structured telemetry data to your configured endpoint

## Prerequisites

For the action to track AI usage, you need to use [git-ai](https://github.com/acunniffe/git-ai) locally to annotate your commits with AI metadata.

## License

MIT License - see [LICENSE](LICENSE) for details.
