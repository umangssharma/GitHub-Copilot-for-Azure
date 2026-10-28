# Workload Context

Workload context is optional authoritative information that Prometheus cannot reliably provide. It may improve recommendation specificity, but it never changes detection evidence.

Create the template with the CLI. Preserve its `schemaVersion`, `workloadId`, and `identity`; edit only `configuration` and `objectives`.

## Supported fields

| Field | Valid value |
|---|---|
| `configuration.modelId` | Non-empty string matching `identity.model_name` |
| `configuration.runtimePrecision` | Non-empty string such as `fp16` or `bf16` |
| `configuration.servingEngine` | Non-empty string such as `vllm` |
| `configuration.servingEngineVersion` | Non-empty string |
| `configuration.tensorParallel` | Integer greater than zero |
| `configuration.maxModelLen` | Integer greater than zero |
| `configuration.maxNumSeqs` | Integer greater than zero |
| `configuration.gpuMemoryUtilization` | Number greater than `0` and at most `1` |
| `configuration.prefixCachingEnabled` | Boolean |
| `configuration.chunkedPrefillEnabled` | Boolean |
| `configuration.replicaCount` | Integer greater than zero |
| `objectives.latencyTargetMs` | Number greater than zero |

## Rules

- Every field is optional. Keep unknown values `null`.
- Never guess from model defaults, generic vLLM defaults, or similarly named workloads.
- Values may come from the user or authoritative workspace configuration.
- Do not copy secrets, tokens, Prometheus endpoints, or kubeconfig data into the context.
- Do not modify generated identity fields.
- Validate the file before passing it to `diagnose` or `recommend`.
- If validation reports a workload ID or identity mismatch, regenerate the context for the selected workload instead of manually forcing the identity.

