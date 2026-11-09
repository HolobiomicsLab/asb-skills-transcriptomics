---
name: cell-cycle-phase-scoring-tricycle
description: Use when you need to evaluate whether forecast or observed single-cell
  transcriptomic states recapitulate cell-cycle progression, by mapping cells (genes
  x cells matrices, e.g. scEU-seq hTERT RPE data) to tricycle phase positions.
when_to_use_negative:
- The cells of interest are quiescent or non-cycling — the 0 to 2π cell-cycle embedding
  presumes constitutively dividing cells like hTERT RPE
- You need mechanistic regulatory network structure rather than a phase-coordinate
  summary; tricycle scoring discards gene-level regulation
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tricycle
  role: Maps predicted or observed single-cell expression states onto the circular
    0 to 2π cell-cycle position embedding used to score forecast ordering
- name: RNAForecaster
  role: Produces the recursively forecast hourly expression states that are subsequently
    scored by tricycle
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Julia 1.6
  role: Runtime environment in which RNAForecaster (Flux 0.13.0, DiffEqFlux 1.46.0)
    generates the forecast states being scored
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:08223b70251125ca392031b7cd517c86f568c5151335444a3c04284f7d155c31
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
  - Julia 1.6
  - RNAForecaster
  - tricycle
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# cell-cycle-phase-scoring-tricycle

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores single-cell expression states, including RNAForecaster recursive forecasts, with the tricycle cell-cycle embedding, which maps each cell to a circular 0 to 2π coordinate so that predicted ordering of cell-cycle progression can be quantified against randomized controls.

## When to use

Use when you need to evaluate whether forecast or observed single-cell transcriptomic states recapitulate cell-cycle progression, by mapping cells (genes x cells matrices, e.g. scEU-seq hTERT RPE data) to tricycle phase positions.

## When NOT to use

- The cells of interest are quiescent or non-cycling — the 0 to 2π cell-cycle embedding presumes constitutively dividing cells like hTERT RPE
- You need mechanistic regulatory network structure rather than a phase-coordinate summary; tricycle scoring discards gene-level regulation

## Inputs

- Predicted or observed log1p-normalized gene x cell expression matrices (e.g. 72 hourly RNAForecaster forecast states from rpeLabeling.h5ad / rpeExampleData.jld2)
- Cell annotations (e.g. labelingTime == 60.0 min subset of Battich et al. scEU-seq hTERT RPE cells)
- Gene names for the filtered (top variance quartile) feature set

## Outputs

- Per-cell tricycle cell-cycle position scores in the 0 to 2π circular embedding for each forecast hour
- Per-hour ordering metrics with randomized controls, written to tricycle_ordering_metrics.csv

## How to apply

After obtaining per-cell expression states — here the 72 hourly RNAForecaster recursive predictions from the 405 scEU-seq 4sU 60-min-labeled hTERT RPE cells (top variance-quartile genes, log1p Float32, ensemble median of 10 neural ODE networks) — apply the tricycle method to embed each predicted cell into the circular cell-cycle coordinate ranging from 0 to 2π. Compute, at each forecast hour, an ordering metric comparing the predicted cell-cycle positions against randomized score controls, writing per-hour results to tricycle_ordering_metrics.csv. Correct application is judged by predictions being significantly more ordered than random at all 72 forecast hours (p < 1e-16 in the source work, Figures 5B and 6A), with the degree of ordering declining per day.

## Related tools

- **tricycle** (Maps predicted or observed single-cell expression states onto the circular 0 to 2π cell-cycle position embedding used to score forecast ordering)
- **RNAForecaster** (Produces the recursively forecast hourly expression states that are subsequently scored by tricycle) — https://github.com/rossinerbe/RNAForecaster.jl
- **Julia 1.6** (Runtime environment in which RNAForecaster (Flux 0.13.0, DiffEqFlux 1.46.0) generates the forecast states being scored)

## Examples

```
t0Estimate = estimateT0LabelingData(labeledData, totalData, unlabeledData, labelingTime)
```

## Evaluation signals

- Each predicted cell is assigned a tricycle position on the 0 to 2π circular scale for all 72 forecast hours
- At every forecast hour the ordering metric shows predicted positions significantly more ordered than randomized score controls (source work: p < 1e-16)
- The degree of ordering declines progressively per forecast day, reproducing Figures 5B and 6A
- The per-hour metric table is complete, with one row per forecast hour (72 rows) saved to tricycle_ordering_metrics.csv

## Limitations

- Ordering quality degrades with forecast horizon, declining per day, so scores at later hours reflect weaker recapitulation of cell-cycle progression
- The training data contained only 405 cells, so rare cell-cycle events may not be captured; a larger training set might allow RNAForecaster to better capture these less common events

## Evidence
