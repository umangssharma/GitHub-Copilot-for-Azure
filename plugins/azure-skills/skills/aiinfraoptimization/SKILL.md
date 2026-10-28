---
name: aiinfraoptimization
description: "Diagnose and optimize already-deployed AI inference workloads through the aiinfraoptimization Azure CLI extension. Copilot runs the workload-intelligence commands, interprets detections, and presents ranked recommendations without applying them. WHEN: \"optimize my AI workload\", \"diagnose my inference workload\", \"find inference performance issues\", \"analyze Prometheus for my model server\", \"why is vLLM unhealthy\", \"recommend changes for my deployed workload\", \"run InfraIQ\", \"AI infrastructure optimization\". DO NOT USE FOR: choosing VMs before deployment, VM SKU sizing, capacity planning for a new workload, or applying recommendation actions."
license: MIT
metadata:
  author: Microsoft
  version: "0.0.0-placeholder"
---

# AI Infrastructure Optimization

Provide a guided Copilot experience over the `az aiinfraoptimization workload-intelligence` CLI. Run the commands for the user, interpret their results, preserve the generated evidence, and explain the next decision.

## Capability Routing

| User intent | Route |
|---|---|
| Diagnose, detect, or optimize an existing workload | Read and follow [workload optimization workflow](workflows/workload-optimization.md) |
| Inspect only detections or replay a historical window | Read and follow [workload optimization workflow](workflows/workload-optimization.md), using its detection-only route |
| Recommend from an existing detection file | Read and follow [workload optimization workflow](workflows/workload-optimization.md), using its recommendation-only route |
| Open the local investigation UI | Read [CLI command reference](references/command-reference.md) and use `toolkit open` |

## Experience Rules

- Execute CLI commands internally when terminal access is available. Do not make the user copy commands unless they request a manual procedure.
- Ask only for required inputs the user has not supplied.
- Check all prerequisites before starting an investigation. Offer to install missing prerequisites when a verified platform package is available; otherwise provide the exact official installation path and stop until verification succeeds.
- Present the complete installation plan and obtain approval before installing, upgrading, removing, or replacing software.
- Before each command, state its immediate purpose in one short sentence.
- Treat detection evidence as authoritative. Workload context may improve recommendations but must not change or replace detector evidence.
- Keep every unknown workload-context value `null`; never infer configuration from generic defaults.
- The workflow is read-only. Recommendations may contain action commands, but neither this skill nor the CLI applies them.
- Never execute a recommendation action without a separate future workflow that validates the target and obtains explicit approval.
- Report command failures and partial detector coverage explicitly. Do not turn an error, skipped detector, or missing context into a recommendation.
