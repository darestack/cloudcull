# Setup Guide: Fork & Deploy

CloudCull is a prototype audit runner. Follow these steps to deploy the simulated or dry-run workflow for review.

## 1. Fork the Repository
Fork this repository to your internal GitHub organization to maintain control over the execution environment.

## 2. Configure GitHub Secrets
Add the following secrets to your forked repository (`Settings > Secrets and variables > Actions`):

| Secret Name | Description |
|-------------|-------------|
| `AWS_ACCESS_KEY_ID` | IAM user or role credential. Read-only access is enough for audits; stop permissions are only needed for reviewed active operations. |
| `AWS_SECRET_ACCESS_KEY` | Corresponding secret key |
| `AZURE_CLIENT_ID` | Service Principal App ID |
| `AZURE_CLIENT_SECRET` | Service Principal Secret |
| `AZURE_TENANT_ID` | Azure Tenant ID |
| `AZURE_SUBSCRIPTION_ID` | Azure Subscription ID |
| `ANTHROPIC_API_KEY` | Required if using Anthropic/Claude |
| `GEMINI_API_KEY` | Required if using Gemini |
| `METRICS_PORT` | Port for Prometheus metrics (Default: 8000) |

## 3. Enable Automation
The audit workflow runs according to the schedule in `.github/workflows/cull_report.yml`.

## 4. Access the Dashboard
Once the first audit completes, your dashboard will be available at:
`https://{your-org}.github.io/cloudcull`

---

> [!TIP]
> **Least Privilege Practice:** Use a dedicated role and start with read-only permissions. Add stop/deallocate permissions only for reviewed active-operation tests.
> - **AWS:** `ec2:DescribeInstances`, `cloudwatch:GetMetricStatistics`, `ec2:StopInstances`
> - **Azure:** `Microsoft.Compute/virtualMachines/read`, `Microsoft.Compute/virtualMachines/deallocate/action`, `Microsoft.Insights/metrics/read`
> - **GCP:** `compute.instances.list`, `monitoring.timeSeries.list`, `compute.instances.stop`, `logging.privateLogEntries.list` (for attribution)
