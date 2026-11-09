---
name: circular-statistics-ordering-assessment
description: Use when recursive neural ODE forecasts (e.g., RNAForecaster 72-step
  predictions) must be validated for biologically ordered progression along a circular
  trajectory such as cell-cycle position from tricycle.
when_to_use_negative:
- Predictions concern a non-cyclic biological process with no circular embedding available
  (tricycle scores only make sense for cell-cycle progression)
- Input is raw counts with no forecast trajectory — there are no sequential hourly
  states to test for ordering
- Only a single predicted time point exists, so no per-hour comparison against randomized
  controls is possible
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: tricycle
  role: Scores each predicted expression state with a circular cell-cycle embedding
    ranging from 0 to 2π, providing the positions used in the ordering metric
- name: RNAForecaster
  role: Produces the recursively forecast hourly expression states that are scored
    and assessed for ordering
  repo: https://github.com/rossinerbe/RNAForecaster.jl
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:c9346afc0dcc5fb746e01f3a4c89c83dbd1ae2b369778d57fcd03cf5ab577a2d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/FertigLab/RNAForecaster.jl
  repo_url: https://github.com/rossinerbe/RNAForecaster.jl
  tools:
  - DiffEqFlux
  - Flux
  - RNAForecaster
  - tricycle
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Circular-statistics ordering assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores forecast or single-cell expression states on a circular 0–2π coordinate (e.g., tricycle cell-cycle position) and tests whether the temporal ordering of predictions exceeds chance by comparing an ordering metric against randomized controls at each forecast hour.

## When to use

Use when recursive neural ODE forecasts (e.g., RNAForecaster 72-step predictions) must be validated for biologically ordered progression along a circular trajectory such as cell-cycle position from tricycle.

## When NOT to use

- Predictions concern a non-cyclic biological process with no circular embedding available (tricycle scores only make sense for cell-cycle progression)
- Input is raw counts with no forecast trajectory — there are no sequential hourly states to test for ordering
- Only a single predicted time point exists, so no per-hour comparison against randomized controls is possible

## Inputs

- Predicted hourly expression matrices from recursive RNAForecaster forecasting (72 one-hour steps, 10-network ensemble median, predictions capped at 2x max training value in log space)
- Gene expression matrix compatible with tricycle cell-cycle scoring
- Gene names/identifier vector for mapping to tricycle

## Outputs

- Per-forecast-hour circular cell-cycle positions in [0, 2π] for each predicted state
- Per-hour ordering metric values compared to randomized controls (tricycle_ordering_metrics.csv)
- Significance assessment (e.g., p < 1e-16 at all hours) and per-day ordering-decline trend

## How to apply

Score each forecast hourly expression state with the tricycle cell-cycle embedding, which maps cells to a circular 0 to 2π coordinate. At each of the forecast hours (e.g., 72 one-hour recursive RNAForecaster steps), compute an ordering metric comparing the predicted cell-cycle positions against randomized score controls, generating a null distribution per hour. Write the per-hour metric values to a results table (e.g., tricycle_ordering_metrics.csv) and assess significance against the randomized null (the scEU-seq RPE reproduction reaches p < 1e-16 at every hour). Interpret trend across hours: a valid forecast shows significant ordering at all hours with the degree of ordering declining per day, matching Figures 5B and 6A.

## Related tools

- **tricycle** (Scores each predicted expression state with a circular cell-cycle embedding ranging from 0 to 2π, providing the positions used in the ordering metric)
- **RNAForecaster** (Produces the recursively forecast hourly expression states that are scored and assessed for ordering) — https://github.com/rossinerbe/RNAForecaster.jl

## Examples

```
ordering_metrics = [computeOrderingMetric(tricycleScore(forecasts[:, h]), randScores) for h in 1:72]; writedlm("tricycle_ordering_metrics.csv", ordering_metrics)
```

## Evaluation signals

- Predicted cell-cycle positions fall within the valid circular range 0 to 2π
- Ordering metric at every one of the 72 forecast hours is significantly more ordered than the randomized controls (reproduction achieves p < 1e-16 at all hours)
- Degree of ordering declines monotonically per day of forecast horizon, matching Figures 5B and 6A
- Per-hour results table (tricycle_ordering_metrics.csv) contains one metric value per forecast hour with no missing hours
- Re-scored t0 input cells yield coherent tricycle positions, confirming the scoring pipeline itself is functional before applying it to forecasts

## Limitations

- The degree of ordering declines per day, so forecasts far beyond the trained interval lose temporal coherence even when still significantly ordered
- Training data contained only 405 cells (60-minute 4sU-labeled scEU-seq cells), and a larger training set might be needed to capture rare cell-cycle events such as checkpoints that ordering metrics will not reveal
- The circular embedding assesses ordering only in cell-cycle position space; it cannot verify accuracy of gene-level expression values outside that trajectory

## Evidence

- [other] Validation criterion of ordering versus random at all hours: [evidence span withheld — anchor status: not_found]
