---
name: total-counts-per-cell-quality-assessment
description: Use when recursively forecasting single-cell expression over many time
  steps (e.g., 72 hourly steps with predictCellFutures) and you must verify the neural
  ODE's predictions have not exploded beyond biologically realistic expression levels.
when_to_use_negative:
- Input is a single one-step prediction rather than recursive multi-step forecasting
  — extreme outputs are not compounded, so the total-counts stability check is less
  informative
- Data are not log-normalized single-cell expression from paired dynamic time points
  (e.g., standard scRNA-seq without metabolic labeling), which RNAForecaster's training
  scheme requires
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: RNAForecaster
  role: Provides predictCellFutures for recursive hourly forecasting whose capped
    outputs are summed and summarized as median total counts per cell
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Flux
  role: Machine-learning package (version 0.13.0) in which the neural ODE underlying
    the forecast is trained
- name: DiffEqFlux
  role: Package (version 1.46.0) used with the Tsit5 ODE solver to train the neural
    ODE
- name: Julia 1.6
  role: Implementation language in which the capping, recursive prediction, and per-cell
    total-count summarization are executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:4b316bb47d10af7e6aaddd55da988f4a1945fe05c9a745d05494311a46f11a8f
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# total-counts-per-cell-quality-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A quality-control check for RNAForecaster's recursive multi-step predictions: after capping expression forecasts at an allowed maximum, per-gene predictions are summed per cell and the median total counts per cell (e.g., after 72 h of cell-cycle forecasting on scEU-seq RPE data) is computed to confirm no runaway growth of predicted expression.

## When to use

Use when recursively forecasting single-cell expression over many time steps (e.g., 72 hourly steps with predictCellFutures) and you must verify the neural ODE's predictions have not exploded beyond biologically realistic expression levels.

## When NOT to use

- Input is a single one-step prediction rather than recursive multi-step forecasting — extreme outputs are not compounded, so the total-counts stability check is less informative
- Data are not log-normalized single-cell expression from paired dynamic time points (e.g., standard scRNA-seq without metabolic labeling), which RNAForecaster's training scheme requires

## Inputs

- Trained RNAForecaster network (e.g., trained on log1p-normalized t0/t1 matrices from scEU-seq RPE cells labeled with 4sU for 60 min, trainingProp = 1.0, learningRate = 0.0001, batchsize = 25)
- Allowed maximum expression cap computed as two times the maximum observed in the training data, in log space
- Capped recursive predictions (72 hourly steps) per gene per cell from predictCellFutures

## Outputs

- Median total counts per cell after 72 h (saved as median_total_counts_72h.csv)
- Qualitative verdict: forecast is stable/non-exploding vs. runaway growth

## How to apply

First apply the expression-prior cap from the trained network: predictions above the allowed maximum (by default two times the maximum observed in the training data, in log space — a 1.2 log-unit increase for the scEU-seq run) are set to the maximum, and negative predictions are clamped to zero. Then perform recursive forecasting with predictCellFutures for 72 hourly steps, feeding each step's output back into the input nodes, keeping the cap active at every recursive step. At the final time point, sum the capped per-gene predictions across all genes for each cell and compute the median total counts per cell. The rationale is that recursive prediction can compound extreme outputs, so the summed per-cell total is a sensitive summary of whether the forecast stays realistic. Correctness is judged by the median total counts per cell at 72 h remaining at a realistic, non-exploding level consistent with the paper's Figure 6B.

## Related tools

- **RNAForecaster** (Provides predictCellFutures for recursive hourly forecasting whose capped outputs are summed and summarized as median total counts per cell) — https://github.com/rossinerbe/RNAForecaster.jl
- **Flux** (Machine-learning package (version 0.13.0) in which the neural ODE underlying the forecast is trained)
- **DiffEqFlux** (Package (version 1.46.0) used with the Tsit5 ODE solver to train the neural ODE)
- **Julia 1.6** (Implementation language in which the capping, recursive prediction, and per-cell total-count summarization are executed)

## Examples

```
using Statistics; preds = predictCellFutures(network, cells, 72); capped = min.(max.(preds, 0.0), maxAllowed); med72 = median(vec(sum(capped, dims=1)))
```

## Evaluation signals

- Median total counts per cell at 72 h remains at a realistic, non-exploding level consistent with the paper's Figure 6B
- No capped prediction exceeds the allowed maximum (two times the maximum observed in the training data, in log space)
- No negative expression values remain in the final predictions (all clamped to zero)
- The cap is applied at every recursive step, not only at the final 72-hour output
- The per-gene predictions are summed across all filtered genes (e.g., top-quartile variance genes in the scEU-seq processing) before taking the per-cell median

## Limitations

- The prior that expression cannot exceed the observed range is enforced post hoc rather than learned — the article notes 'future work may be needed to teach the neural network about the prior probability for gene expression levels outside the range normally observed in cells, rather than having to enforce this prior'
- A stable median total count does not guarantee correct gene-level dynamics; the paper notes the training data contained only 405 cells, which may cause RNAForecaster to miss rare cell-cycle events
- The check is specific to recursive multi-step prediction; a one-step forecast does not compound extreme values in the same way

## Evidence
