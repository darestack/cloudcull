# Terraform State Semantics

CloudCull can generate and, behind `--active-ops --no-dry-run`, execute Terraform state-removal actions. This document explains the boundary clearly.

## What `terraform state rm` Does

`terraform state rm <address>` removes the resource address from Terraform's state file. Terraform will no longer track that object as managed by the current workspace.

## What It Does Not Do

`terraform state rm` does not destroy, stop, deallocate, or resize the real cloud resource. If the cloud resource is still running, it can continue to incur cost after state removal.

## CloudCull's Prototype Flow

For active operations, CloudCull attempts this order:

1. Stop or deallocate the cloud resource through the provider SDK.
2. Resolve the Terraform address from state by matching the provider resource type and physical ID.
3. Create a local `terraform.tfstate.backup.<timestamp>` file when a local state file exists.
4. Run `terraform state rm <address>` with `subprocess.run(..., shell=False)`.

If CloudCull cannot find the resource in Terraform state, it logs a state-sync failure and skips state removal for that resource.

## Production Requirements Before Use

Do not use this flow as production automation until the environment has been reviewed for:

- remote backend configuration and locking behavior;
- concurrent Terraform or CloudCull runners;
- permissions needed for stop/deallocate and state access;
- recovery steps for false positives;
- drift detection after state removal;
- import or reconciliation steps when a resource should return to Terraform management;
- explicit approval and audit logging around every active operation.

## Interview-Safe Explanation

"Removing a resource from Terraform state changes Terraform's tracking, not the cloud resource itself. CloudCull's prototype active flow attempts to stop or deallocate the cloud resource first, then remove the Terraform state address if it can safely resolve it. In production, I would require backend locking, environment-specific validation, and human approval before enabling that path."
