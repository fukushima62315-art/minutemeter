# MinuteMeter — GitHub Actions cost per job, on every PR

See what your GitHub Actions runs actually cost. MinuteMeter attributes a USD cost to each job in a workflow run and posts the breakdown as a PR comment and job summary.

## Key features

- **Per-job cost attribution**: Breaks down workflow run costs by job, detecting runner type (Linux, Windows, macOS, self-hosted) and core count automatically.
- **Budget alerts**: Flags runs when total cost exceeds a configurable USD threshold.
- **PR comments and job summaries**: Posts cost breakdown directly to PRs and workflow summaries—no external dashboard or sign-up required.
- **Billing-accurate**: Computes cost as per-job duration × runner list price, billed per started minute (rounded up), matching GitHub's billing model.
- **Zero external data**: Runs inside your CI; no data leaves GitHub.
- **Graceful degradation**: Never fails your CI; API or permission errors downgrade to warnings.

## Technology stack

- **Language**: Python 3 (stdlib only, no external dependencies)
- **Delivery**: GitHub Actions
- **APIs**: GitHub REST API (actions, pull-requests endpoints)

## Installation / setup

Add a workflow file (e.g., `.github/workflows/minutemeter.yml`) to your repository:

```yaml
name: MinuteMeter
on:
  workflow_run:
    workflows: ["CI"]        # the workflow you want to price
    types: [completed]

permissions:
  actions: read              # read run/job timing
  pull-requests: write       # comment the breakdown

jobs:
  cost:
    runs-on: ubuntu-latest
    steps:
      - uses: minutemeter/minutemeter@v1
        with:
          budget_usd: "1.00"   # optional: flag runs over $1.00
```

**Required permissions:**
- `actions:read` — to read workflow run and job timing
- `pull-requests:write` — to comment the cost breakdown on PRs

**Inputs:**

| Input | Default | Description |
|---|---|---|
| `github_token` | `${{ github.token }}` | GitHub API token with required scopes. |
| `budget_usd` | (none) | Optional: flag the run when its cost exceeds this USD amount. |
| `run_id` | triggering run | Workflow run ID to analyze (auto-detected from event). |
| `pr_number` | auto | PR number to comment on (auto-detected from the event). |

## Usage

Once the workflow is configured, MinuteMeter runs automatically after your CI completes. On each run, you will receive a PR comment like:

```
### 💸 MinuteMeter — GitHub Actions cost
**Run total: $0.4820** (impactful jobs first) · 🔴 OVER budget $1.00? no

| Job | Runner | Min | $ |
|---|---|--:|--:|
| integration (macos) | macos | 7 | $0.4340 |
| build | linux | 8 | $0.0480 |
| lint | linux | 1 | $0.0060 |
```

**Cost calculation:**
- Per-job duration is multiplied by the runner's list price per minute.
- Billing rounds up to the nearest started minute.
- Runner type is detected from job labels (`ubuntu-latest`, `windows-latest`, `macos-latest`, self-hosted, or larger runners like `ubuntu-latest-8-cores`).
- **Rates (USD/minute)**: Linux $0.006, Windows $0.010, macOS $0.062, self-hosted currently $0.000 (the announced $0.002/min charge was shelved).

## Directory structure

```
.
├── README.md                    # Project overview and quick start
├── src/
│   ├── minutemeter.py          # Main cost computation and API integration
│   └── rates.py                # GitHub Actions per-minute runner rates (USD)
└── tests/
    └── test_cost_engine.py     # Unit tests for cost computation
```