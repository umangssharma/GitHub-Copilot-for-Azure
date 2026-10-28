# Troubleshooting

| Symptom | Response |
|---|---|
| `az` is unavailable | Stop and ask the user to install Azure CLI |
| Extension command is unknown | Check `az extension show --name aiinfraoptimization`; offer installation with approval |
| Prometheus returns `401` or `403` | Confirm `az account show`, intended subscription, `Monitoring Data Reader`, and role propagation |
| Endpoint is unreachable | Verify the Prometheus query endpoint and TLS/network access; do not continue to diagnosis |
| Preflight is `NOT_READY` | Report missing required metrics or labels and ask whether to continue with partial coverage |
| No workloads are discovered | Confirm the exact endpoint and time window; widen only with user agreement |
| No detections are returned | Report whether coverage was complete; absence of detections with partial coverage is inconclusive |
| Context validation fails | Fix schema, ranges, or identity; never bypass validation |
| Recommendation count is zero | Distinguish no actionable evidence, missing workload context, and recommendation-service failure |
| Recommendation API fails | Preserve `detections.json`, report the error, and offer a later retry without rerunning detection |

## Reinstall the extension

Removing or replacing an extension changes the user's environment. Obtain approval first:

```powershell
az extension remove --name aiinfraoptimization
```

Then follow [prerequisites](prerequisites.md) to install the supported version.

## Safety

- Never expose an access token.
- Never treat a skipped detector as healthy.
- Never claim no issue was found unless detector coverage was complete.
- Never execute commands contained in recommendation `Actions`.
- Never rerun detection merely to add workload context; reuse the persisted evidence.
