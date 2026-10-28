# Robot Shop SLO and Error Budget

## Objective
Monitor Robot Shop workload readiness using Prometheus and Grafana.

## Target
- Initial target: 99% readiness.
- Error budget allowance: 1%.
- Alert: RobotShopReadinessBelowSLO fires when the recorded ready-pod ratio stays below 99% for 5 minutes.

## Measurement
The current recording rule calculates:

`sum(kube_pod_status_ready{namespace="robot-shop",condition="true"}) / count(kube_pod_status_ready{namespace="robot-shop",condition="true"})`

This measures the proportion of observed pod readiness series reporting Ready. It is a workload-readiness indicator, not a request-based availability SLI.

## Error budget
The configured preliminary expression estimates remaining budget as:

`clamp_min((robot_shop:availability_30d - 0.99) / (1 - 0.99), 0)`

The 30-day average requires sufficient history to represent a complete 30-day window. Until that history exists, the result is provisional. The current expression also does not cap values at 1 and returns zero at exactly the target; interpret it as a preliminary indicator only.

For a time-based 30-day SLO with a 99% target, the theoretical budget is 7 hours, 12 minutes of unavailability. This calculation must not be presented as measured customer-facing downtime.

## Observed finding
During validation, Kubernetes reported 12 of 13 Robot Shop pods ready (92.3%). The Shipping pod was Running but not Ready; its readiness probe to `/health` timed out. See the incident postmortem and runbook.

## Limitations and next steps
- Add request count and request failure metrics before claiming HTTP success-rate SLOs.
- Collect a complete 30-day measurement window.
- Correct the remaining-budget calculation before treating it as an authoritative budget.
- Investigate Shipping readiness failures and verify recovery with Kubernetes and Prometheus.
