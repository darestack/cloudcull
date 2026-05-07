# CloudCull

> Cloud cost audit prototype with dry-run reporting, provider adapters, remediation manifests, and a dashboard demo.

[![Audit Workflow](https://github.com/daretechie/cloudcull/actions/workflows/cull_report.yml/badge.svg)](https://github.com/daretechie/cloudcull/actions/workflows/cull_report.yml)
[![Live Dashboard](https://img.shields.io/badge/dashboard-demo-blue)](https://daretechie.github.io/cloudcull/)
[![License: MIT](https://img.shields.io/badge/license-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

CloudCull explores a practical DevOps problem: cloud resources, especially GPU instances, can keep running after they are no longer useful. The safe first step is not automatic deletion. It is a reviewable audit report that shows what was found, why it looks wasteful, and what action should be taken next.

## Current Scope

This repository is a prototype, not a production FinOps platform.

It is designed to:

- run cost-audit checks from a CLI or GitHub Actions workflow;
- support dry-run reporting before any active operation;
- separate provider adapters from reporting logic;
- write remediation manifests for human review;
- show audit output in a lightweight dashboard demo.

It is not claiming:

- autonomous production remediation;
- guaranteed savings;
- real-time multi-cloud governance;
- enterprise-grade controls;
- deletion or stop actions without review.

## Architecture

```mermaid
graph LR
    Trigger["CLI / GitHub Action"] --> Probe["Provider adapter"]
    Probe --> Analyze["Audit rules"]
    Analyze --> Report["Dry-run report"]
    Report --> Manifest["Remediation manifest"]
    Manifest --> Review["Human review"]
    Review --> Action["Optional active operation"]
```

## Tech Stack

| Area | Tools |
|---|---|
| CLI / audit logic | Python |
| Cloud adapters | AWS, Azure, GCP-oriented modules |
| Automation | GitHub Actions |
| Packaging | Docker, Docker Compose |
| Dashboard | React / Vite |
| Security check | Bandit |

## How to Run

```bash
git clone https://github.com/daretechie/cloudcull.git
cd cloudcull
uv sync
./scripts/demo.sh
```

Docker:

```bash
docker build -f infra/Dockerfile -t cloudcull .
docker run --env-file .env cloudcull --simulated --dry-run
```

## Active Operations Boundary

Use active operations only after a dry run has been reviewed.

```bash
uv run cloudcull --region us-east-1 --active-ops
uv run cloudcull --platform azure --active-ops
uv run cloudcull --platform gcp --active-ops
```

Before using active operations in a real account, add:

- a least-privilege IAM or cloud-provider policy;
- a redacted sample report;
- an approval step;
- a rollback or recovery note;
- logs proving exactly what action was taken.

## Evidence To Add

| Evidence | Status |
|---|---|
| Dry-run output with redacted cloud fixture | Needed |
| Example remediation manifest | Needed |
| GitHub Actions audit run screenshot | Needed |
| Least-privilege policy for each provider | Needed before production claims |
| Dashboard screenshot | Demo available, screenshot should be committed |

## Repository Orientation

| Directory | Purpose |
|---|---|
| `src/` | Python modules, adapters, and CLI entrypoint |
| `infra/` | Docker and deployment-related files |
| `config/` | Configuration and generated manifests |
| `scripts/` | Demo and utility scripts |
| `docs/` | Architecture, setup, operations, and security notes |
| `tests/` | Unit and integration tests |
| `dashboard/` | Dashboard demo |

## Security Checks

```bash
uv run bandit -c pyproject.toml -r .
```

## Notes For Reviewers

This project is intentionally framed as a prototype. The important engineering signal is the safety model: dry-run first, manifest output, human review, and explicit limits around active operations.
