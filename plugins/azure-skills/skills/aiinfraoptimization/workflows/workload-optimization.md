# Workload Optimization Workflow

Diagnose an already-deployed inference workload from historical Prometheus telemetry and obtain ranked, read-only recommendations.

## Quick Reference

| Property | Value |
|---|---|
| Inputs | Prometheus query endpoint, UTC or relative time window, optional subscription and workload ID |
| Primary command | `az aiinfraoptimization workload-intelligence diagnose` |
| Artifacts | `detections.json`, `recommendations.json`, optional `workload-context.json` |
| Mutations | None; recommendation actions are displayed but never applied |
| Parent | [aiinfraoptimization](../SKILL.md) |

## When to Use

Use this workflow for running workloads when the user wants to discover workloads, investigate inference behavior, replay detectors, understand findings, or receive optimization recommendations.

Do not use it for pre-deployment VM selection, capacity planning for a new workload, or executing a recommended change.

## Required Inputs

Ask only for values not already supplied:

| Input | Requirement |
|---|---|
| Prometheus endpoint | Required; use the query endpoint, not the Azure portal resource URL |
| Time window | Required; accept `--since` or an explicit UTC `--from` and `--to` pair |
| Subscription | Optional unless the expected Azure identity or subscription is ambiguous |
| Workload ID | Optional; discover it when absent |
| Workload context | Optional; collect only authoritative configuration and objectives |
| Output location | Optional; default to `.infraiq/runs/<UTC timestamp>/` in the workspace |

## Workflow

### 1. Check and prepare prerequisites

Read and follow [prerequisites](../references/prerequisites.md). Check every required executable, offer supported installations as one approval-gated plan, and verify each installed component. Do not query Prometheus until Azure CLI authentication, extension availability, and the intended subscription are confirmed.

### 2. Establish the run

1. Normalize the requested window:
   - Preserve explicit UTC timestamps.
   - For a relative duration, use `--since <duration>`.
   - Never silently replace a historical window with the current time.
2. Create `.infraiq/runs/<UTC timestamp>/` unless the user supplied an output directory.
3. Retain the endpoint, window, workload ID, and artifact paths for all later commands in this run.

### 3. Run preflight

Run `preflight` with the effective endpoint and window. Use the exact template in [CLI command reference](../references/command-reference.md).

- `READY` may include failed optional checks; explain them without treating the run as failed.
- `NOT_READY` means required detector inputs are missing. Report the missing coverage and ask whether to continue with partial coverage.
- Authentication or reachability failures must be resolved before continuing.

### 4. Discover and select the workload

Run `workload discover` with the same endpoint and window.

- If no workload is returned, stop and use [troubleshooting](../references/troubleshooting.md).
- If exactly one workload is returned, state that it was selected.
- If multiple workloads are returned and no workload ID was supplied, ask the user to select one.
- Never invent a workload ID from a pod name, model name, or namespace.

### 5. Diagnose

Run `diagnose` with the same endpoint, window, selected workload ID, and run directory.

- If an authoritative context file was supplied, validate it first and pass it with `--context`.
- Use table output for the immediate operational summary.
- Preserve `detections.json` and `recommendations.json` from the run directory.
- Do not add `--include-diagnostics` unless the user or support specifically asks for diagnostic artifacts.

Interpret the result:

1. Summarize detector coverage, including skipped detectors and why they were skipped.
2. List each episode with detector, severity, state, duration, confidence, and diagnosis.
3. Distinguish a missing-context issue from a recommendation.
4. If there are no detections, say whether coverage was complete before concluding that nothing fired.

### 6. Enrich recommendations when context is missing

If diagnosis found actionable evidence but reports missing workload context:

1. Explain which authoritative fields would improve the recommendation.
2. Ask whether the user wants to provide them.
3. If yes, read [workload context](../references/workload-context.md).
4. Run `context create` for the selected workload and the same endpoint and window.
5. Update only `configuration` and `objectives` with values the user supplied or that are authoritative in the workspace. Leave every unknown value `null`.
6. Run `context validate`. Fix validation failures before continuing.
7. Run `recommend` with the existing `detections.json` and validated context. Do not rerun detection.

### 7. Present recommendations

For each ranked recommendation, report:

- Finding and recommendation title.
- Impact, effort, and confidence.
- Evidence and rationale from the complete JSON artifact.
- Proposed actions and verification steps.
- Any unmapped evidence, missing context, or service failure.

Clearly label action commands as proposals. Do not execute them.

Finish with the paths of the generated artifacts and offer the next read-only option: inspect complete evidence, provide more context, replay a different window, or open the toolkit UI.

## Alternate Routes

| Intent | Route |
|---|---|
| Detection only | Run `replay`; do not call `recommend` |
| Existing `detections.json` | Validate the file belongs to one workload, then run `recommend` |
| Context preparation only | Run `context create`, update known values, then `context validate` |
| Inspect detector catalog | Use `detector list` or `detector show` |
| Local UI investigation | Run `toolkit open` |

Use the exact templates in [CLI command reference](../references/command-reference.md).

## Error Handling

Use [troubleshooting](../references/troubleshooting.md). Stop on an invalid context, wrong workload identity, authentication failure, or unreachable endpoint. Partial detector coverage may continue only after it is clearly reported.

> **Return to parent:** When the requested investigation is complete, return control to the `aiinfraoptimization` skill.
