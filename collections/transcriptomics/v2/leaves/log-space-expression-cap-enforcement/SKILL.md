---
name: log-space-expression-cap-enforcement
description: Use when performing recursive (multi-step) neural ODE expression forecasting
  with RNAForecaster, e.g. predicting scEU-seq RPE cell-cycle gene expression over
  72 hourly steps, to prevent extreme predictions.
when_to_use_negative:
- Standard scRNA-seq protocols without paired time-point measurements of the same
  cell — RNAForecaster training requires dynamic (e.g. metabolic labeling) data, so
  there are no recursive predictions to cap.
- Single one-step predictions that stay within the training expression distribution
  and do not feed outputs back into the network inputs — no extreme-prediction control
  loop is needed.
- You need a learned prior on expression outside the observed range — the cap is enforced
  post hoc, not learned, so it is the wrong mechanism for probabilistic extrapolation.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: RNAForecaster.jl
  role: Provides predictCellFutures and the built-in allowed-maximum capping of expression
    predictions during recursive forecasting
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Julia 1.6
  role: Runtime environment in which the capped recursive prediction workflow is executed
- name: Flux 0.13.0 / DiffEqFlux 1.46.0
  role: Machine-learning packages used to train the underlying neural ODE (Tsit5 solver)
    whose recursive outputs are capped
  repo: https://github.com/FertigLab/RNAForecasterPaperCode
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:0696ef04c26a88b55014e0a9d20ae173f13d421e07b513b0a6ce03a5531d5eea
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
  - Flux 0.13.0 / DiffEqFlux 1.46.0
  - Julia 1.6
  - RNAForecaster
  - RNAForecaster.jl
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# log-space-expression-cap-enforcement

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Enforces a realistic prior on expression bounds during RNAForecaster's recursive predictions by capping predicted expression at the allowed maximum (default two times the maximum observed in the training data, in log space) and clamping negative predictions to zero, preventing runaway growth over multi-step forecasts.

## When to use

Use when performing recursive (multi-step) neural ODE expression forecasting with RNAForecaster, e.g. predicting scEU-seq RPE cell-cycle gene expression over 72 hourly steps, to prevent extreme predictions.

## When NOT to use

- Standard scRNA-seq protocols without paired time-point measurements of the same cell — RNAForecaster training requires dynamic (e.g. metabolic labeling) data, so there are no recursive predictions to cap.
- Single one-step predictions that stay within the training expression distribution and do not feed outputs back into the network inputs — no extreme-prediction control loop is needed.
- You need a learned prior on expression outside the observed range — the cap is enforced post hoc, not learned, so it is the wrong mechanism for probabilistic extrapolation.

## Inputs

- Trained RNAForecaster network (Julia/Flux/DiffEqFlux neural ODE) trained on log1p-normalized count matrices
- Log1p-normalized t0 expression matrix (input layer values) from the scEU-seq RPE example (rpeExampleData.jld2)
- Training-data log-space expression distribution used to compute the 2x maximum cap (1.2 log-unit increase for the scEU-seq run)

## Outputs

- 72-hour recursively predicted, capped per-gene expression matrix (cap applied at every recursive step)
- Median total (summed per-gene) counts per cell at 72 h, e.g. written to median_total_counts_72h.csv

## How to apply

After training the neural ODE on log1p-normalized t0/t1 scEU-seq matrices (e.g. 4sU 60-min labeled RPE cells, top-quartile variance genes), compute the allowed maximum as two times the maximum observed in the training data in log space. Run recursive forecasting with predictCellFutures for 72 hourly steps, feeding each step's output back into the input nodes. At every recursive step, set predictions above the allowed maximum to the maximum and clamp negative predictions to zero. Rationale: the neural ODE is not constrained to the observed expression space because it takes input in gene-dimensional space, so the cap enforces post hoc the prior that expression cannot exceed realistic bounds derived from the training distribution.

## Related tools

- **RNAForecaster.jl** (Provides predictCellFutures and the built-in allowed-maximum capping of expression predictions during recursive forecasting) — https://github.com/rossinerbe/RNAForecaster.jl
- **Julia 1.6** (Runtime environment in which the capped recursive prediction workflow is executed)
- **Flux 0.13.0 / DiffEqFlux 1.46.0** (Machine-learning packages used to train the underlying neural ODE (Tsit5 solver) whose recursive outputs are capped) — https://github.com/FertigLab/RNAForecasterPaperCode

## Examples

```
futures = predictCellFutures(network, t0Log1p, 72, allowedMax = 2 * maximum(trainLog1p))  # capped recursive 72-step forecast
```

## Evaluation signals

- Median total counts per cell after 72 recursive hourly steps remains at a realistic, non-exploding level consistent with the paper's Figure 6B (no runaway growth of predicted expression).
- No predicted expression value exceeds 2x the maximum observed in the training data in log space (i.e. a 1.2 log-unit increase for the scEU-seq run).
- No negative expression values remain in the capped prediction matrix.
- Capped per-gene predictions at 72 h sum to total counts per cell that match the saved median_total_counts_72h.csv.
- Recapitulated cell-cycle gene expression changes remain biologically consistent when scored with the tricycle embedding (0 to 2π).

## Limitations

- The prior on expression outside the observed range is not learned by the network; it is enforced post hoc, and the authors note future work is needed to teach the network this prior instead.
- Training on only 405 scEU-seq cells may fail to capture rare cell-cycle events, and the current architecture may lack the capability to predict cell-cycle checkpoints or rare regulatory exceptions even with the cap in place.
- The limits of predictability over diverse timescales and biological conditions have not been evaluated, so the cap's sufficiency outside 72 h cell-cycle settings is untested.

## Evidence
