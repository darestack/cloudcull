# CloudCull Operations Guide

## Automated Audits (GitHub Actions)
The functionality is defined in `.github/workflows/cull_report.yml`.
- **Frequency**: Hourly (`cron: '0 * * * *'`).
- **Trigger**: Push to `main` or Manual Dispatch.
- **Output**: Deploys to `gh-pages` branch.

### Required GitHub Secrets
To run a real multi-cloud audit, add these to **Settings > Secrets and variables > Actions**:
- `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`
- `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`
- `GOOGLE_CREDENTIALS_JSON` (Base64 encoded SA key), `GOOGLE_CLOUD_PROJECT`
- `ANTHROPIC_API_KEY` (or other AI keys)

## Runbooks

### S1: Dashboard Not Updating
1.  Check **Actions** tab in GitHub.
2.  If failed, check the logs.
3.  Common cause: Expired API Key or missing `simulated` flag in workflow.

### S2: "Pages Not Found" Error
1.  Go to **Settings > Pages**.
2.  Ensure **Source** is set to **GitHub Actions**.

### S3: Dashboard Review Workflow
The dashboard is a review surface for generated audit reports:
1.  **Analyze**: Review the reasoning on each card to understand why the instance was flagged.
2.  **Verify**: Check the metrics (CPU/Network) and owner attribution.
3.  **Review**: Inspect the copied remediation command before running anything manually.
4.  **Execute manually only when approved**: `terraform state rm` removes Terraform tracking. It does not destroy or stop the cloud resource by itself.

### S4: ActiveOps (Automated Remediation)
ActiveOps is a prototype path for explicit, reviewed remediation:
1.  **Analyze**: Run `uv run cloudcull --active-ops --no-dry-run`.
2.  **Verify**: CloudCull will generate `remediation_manifest.json` and show the selected candidates.
3.  **Approve**: The CLI prompts for confirmation before executing stop/deallocate plus state removal.
4.  **Safety**: Before local state removal, CloudCull creates a backup when `terraform.tfstate` exists: `terraform.tfstate.backup.<timestamp>`.
5.  **Audit**: Review the `remediation_manifest.json` for a post-mortem of all actions taken.

Before production use, validate backend locking, remote state access, provider permissions, false-positive handling, and environment-specific rollback steps. Do not use `--auto-approve` outside controlled tests or demos.

### S5: Monitoring Platform Health (Prometheus)
CloudCull exposes real-time metrics for integration with Grafana or Datadog:
1.  **Endpoint**: `http://localhost:8000/metrics` (configurable via `METRICS_PORT`).
2.  **Key Gauges**:
    - `cloudcull_zombies_found_total`: Current count of flagged review candidates.
    - `cloudcull_potential_savings_usd`: Projected monthly savings from pending remediation.

### S6: Partial Remediation Failure
1.  **Scenario**: Physical stop succeeds, but Terraform state removal fails.
2.  **Detection**: CLI logs: "Terraform Execution Failed for [ID]".
3.  **Action**: Review whether the resource should remain in Terraform, be re-imported, or be removed from state.
4.  **Verification**: Re-run the audit and inspect both cloud state and Terraform state.

### S5: Active Operations Dry Run
To exercise the flow without cost:
1.  Run locally with `--dry-run`.
2.  Verify the report and remediation manifest.
3.  Only after review, run with `--active-ops --no-dry-run` in a controlled environment.

### S7: Accessing Secure Logs
Since logs are stored in `logs/audit.log`, they are not accessible via standard static web servers.
1.  **Requirement**: Start the Secure Dashboard Server: `python3 src/dashboard_server.py`.
2.  **Access**: View logs via the dashboard at `http://localhost:8080`.
3.  **Manual View**: Using `tail -f logs/audit.log` on the host machine.

## Security
- **Least Privilege**: The runner only needs `ReadOnly` access for `scan()`.
- **Write Access**: `stop_instance()` requires `ec2:StopInstances` (AWS), `Microsoft.Compute/virtualMachines/deallocate/action` (Azure), `compute.instances.stop` (GCP), etc.
- **Log Isolation**: Logs are kept in a separate directory (`logs/`) and served via an API layer to prevent accidental exposure of sensitive metadata that may be present in debug logs.
- **IaC Integrity**: Remediation actions are dynamically looked up in the Terraform state to prevent state corruption.
- **State Semantics**: `terraform state rm` changes Terraform tracking. It does not stop, delete, or deallocate resources by itself.
