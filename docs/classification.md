# CloudCull Classification Model

CloudCull treats classification as a review aid. A `ZOMBIE` result means "candidate for investigation", not "safe to delete".

## Operational Meaning of `ZOMBIE`

A resource is a zombie candidate when the available signals suggest it may be idle or abandoned:

- very low CPU utilization during the sampled window;
- low network activity;
- no obvious metadata that explains active batch, training, or service work;
- ownership or launch attribution that can be reviewed by a human.

The exact signals depend on what each provider adapter can collect. Missing or stale metrics should lower confidence, not increase automation.

## Current Decision Flow

1. Provider adapters discover candidate GPU instances and collect metrics/metadata.
2. The runner sends sanitized metrics and metadata to the configured analysis provider.
3. The provider returns structured JSON with a decision, reasoning, and confidence.
4. The runner renders the decision, estimates savings when pricing is known, and writes a report.
5. If the decision is `ZOMBIE`, CloudCull generates a remediation manifest for review.

## Evaluation Status

The current repository has unit tests for provider parsing, simulated classification, remediation planning, and active-operation orchestration. That is useful prototype coverage, but it is not enough to claim production classifier quality.

Before relying on the classifier in production, add:

- a labeled dataset of idle and active instances across AWS, Azure, and GCP;
- precision, recall, and false-positive tracking by provider and instance family;
- regression tests for edge cases such as bursty workloads, paused training jobs, scheduled batch work, and missing metrics;
- confidence thresholds that block active operations when evidence is incomplete;
- human sign-off for any resource without strong attribution and metric history.

## Hallucination Controls

CloudCull should treat model output as untrusted until checked against deterministic data. The current code asks providers for structured JSON and falls back to `UNKNOWN` on analysis errors. A production path should also require deterministic guardrails, for example:

- minimum metric-window length;
- required owner or service tags;
- provider API confirmation immediately before remediation;
- explicit "do not stop" tag support;
- a reviewed allowlist of resource types that can enter active operations.
