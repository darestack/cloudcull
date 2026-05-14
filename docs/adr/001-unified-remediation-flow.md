# ADR-001: Unified Remediation Flow (Physical Stop + State Removal)

## Status
Accepted

## Context
CloudCull originally generated Terraform-oriented state-removal actions as the primary remediation output. That can be useful for review, but relying on `terraform state rm` alone creates a critical failure mode: the instance can keep running and incurring costs while Terraform stops tracking it.

## Decision
CloudCull will use a **Unified Remediation Flow** for explicit `ActiveOps` runs. During an `ActiveOps` cycle, the system will:
1.  **Issue a Cloud-Native Stop Command**: Using the respective cloud provider API (Boto3, Azure SDK, Google Client) to deallocate or stop the resource physically.
2.  **Perform Terraform State Removal**: Only if at least one physical stop/deallocate action succeeds, the system attempts to remove matching resource addresses from Terraform state.

## Rationale
- **Cost Control**: Attempts to stop costs for approved idle-resource candidates.
- **Safety**: Stopping/Deallocating is less destructive than Termination/Deletion, allowing for easier rollback if a false positive occurs.
- **State Clarity**: Avoids presenting `terraform state rm` as if it destroys the cloud resource.

## Consequences
- **Permission Requirements**: The service account/role running CloudCull now requires write permissions (e.g., `ec2:StopInstances`).
- **Complexity**: The orchestrator must now track physical stop status before attempting state manipulation.
- **Operational Review**: Production use still requires backend locking, environment-specific validation, false-positive handling, and human approval.
