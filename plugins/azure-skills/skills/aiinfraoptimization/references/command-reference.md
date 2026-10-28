# CLI Command Reference

Use one consistent Prometheus endpoint and time window throughout an investigation.

## Preflight

```powershell
az aiinfraoptimization workload-intelligence preflight `
  --prometheus "<prometheus-query-endpoint>" `
  --from "<utc-start>" `
  --to "<utc-end>" `
  --output table
```

For a relative window, replace `--from` and `--to` with `--since "<duration>"`.

## Discover workloads

```powershell
az aiinfraoptimization workload-intelligence workload discover `
  --prometheus "<prometheus-query-endpoint>" `
  --from "<utc-start>" `
  --to "<utc-end>" `
  --output table
```

## Unified diagnosis

```powershell
az aiinfraoptimization workload-intelligence diagnose `
  --prometheus "<prometheus-query-endpoint>" `
  --from "<utc-start>" `
  --to "<utc-end>" `
  --workload-id "<workload-id>" `
  --output-directory "<run-directory>" `
  --output table
```

Add `--context "<context-file>"` only after successful validation. Add `--include-diagnostics` only when support requests the sanitized run manifest and recommendation request.

## Workload context

```powershell
az aiinfraoptimization workload-intelligence context create `
  --prometheus "<prometheus-query-endpoint>" `
  --from "<utc-start>" `
  --to "<utc-end>" `
  --workload-id "<workload-id>" `
  --output-file "<run-directory>/workload-context.json" `
  --output table

az aiinfraoptimization workload-intelligence context validate `
  --file "<run-directory>/workload-context.json" `
  --output table
```

## Recommend from saved evidence

```powershell
az aiinfraoptimization workload-intelligence recommend `
  --detection-file "<run-directory>/detections.json" `
  --context "<run-directory>/workload-context.json" `
  --output-file "<run-directory>/recommendations-with-context.json" `
  --output table
```

Omit `--context` when no authoritative context is available.

## Detection-only replay

```powershell
az aiinfraoptimization workload-intelligence replay `
  --prometheus "<prometheus-query-endpoint>" `
  --from "<utc-start>" `
  --to "<utc-end>" `
  --workload-id "<workload-id>" `
  --output-file "<run-directory>/detections.json" `
  --output table
```

## Detector catalog

```powershell
az aiinfraoptimization workload-intelligence detector list --output table
az aiinfraoptimization workload-intelligence detector show `
  --detector-id "<detector-id>" `
  --output table
```

## Toolkit UI

```powershell
az aiinfraoptimization workload-intelligence toolkit open
```

Use table output for concise progress and JSON artifacts for complete evidence, coverage, rationale, actions, and verification steps.
