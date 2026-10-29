---
name: negative-prediction-clamping-during-recursion
description: Use when running recursive multi-step predictions with RNAForecaster's
  predictCellFutures (e.g., 72 hourly steps on scEU-seq RPE data) and network outputs
  must be constrained to biologically realistic expression ranges.
when_to_use_negative:
- When making single-step (non-recursive) predictions where outputs are not fed back
  into the network and instability cannot compound
- When the analysis goal is to let the network itself learn the prior probability
  of expression outside the observed range rather than enforce it post hoc
- When input expression values are not in log space, since the default cap is defined
  as two times the maximum observed in the training data in log space
tools:
- name: RNAForecaster
  role: Provides predictCellFutures recursive prediction with built-in max-capping
    and negative-clamping of expression predictions
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: Flux
  role: Machine-learning framework used to train the neural ODE whose recursive outputs
    are capped/clamped (version 0.13.0)
- name: DiffEqFlux
  role: Neural ODE training framework (with Tsit5 solver) producing the network whose
    forecasts are filtered (version 1.46.0)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:d750f44fae730145fa7a63a54c4510349ce953e0c364cc82bb830e76bca47559
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# negative-prediction-clamping-during-recursion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A bound-enforcement step in RNAForecaster recursive forecasting that clamps negative predictions to zero and caps predictions above an allowed maximum (by default two times the maximum observed in the training data, in log space), preventing expression blow-up over repeated feed-back steps such as the 72-hour scEU-seq cell-cycle forecast.

## When to use

Use when running recursive multi-step predictions with RNAForecaster's predictCellFutures (e.g., 72 hourly steps on scEU-seq RPE data) and network outputs must be constrained to biologically realistic expression ranges.

## When NOT to use

- When making single-step (non-recursive) predictions where outputs are not fed back into the network and instability cannot compound
- When the analysis goal is to let the network itself learn the prior probability of expression outside the observed range rather than enforce it post hoc
- When input expression values are not in log space, since the default cap is defined as two times the maximum observed in the training data in log space

## Inputs

- Trained RNAForecaster neural ODE network (trained with Flux.jl/DiffEqFlux.jl, Tsit5 solver) on log1p-normalized t0/t1 scEU-seq RPE matrices
- Log1p-normalized training expression matrix used to compute the 2x-maximum cap in log space
- predictCellFutures recursive forecast horizon (e.g., 72 hourly steps)

## Outputs

- Per-gene, per-cell capped and clamped predicted expression values at each recursive time step
- 72-hour future expression predictions whose per-gene values never exceed the allowed maximum or fall below zero
- Median total counts per cell after 72 hours (e.g., saved to median_total_counts_72h.csv)

## How to apply

Compute the allowed maximum expression cap as two times the maximum observed in the training data in log space (corresponding to a 1.2 log-unit increase for the scEU-seq run). At every recursive step of predictCellFutures—where each step's output is fed back into the input nodes—set any predicted gene expression above the allowed maximum to the maximum, and clamp negative predictions to zero. This enforces a realistic prior on the upper bound of expression given the training distribution, so that median total counts per cell after 72 hours remain at a realistic, non-exploding level. Apply the clamping inside the prediction loop, not as a post hoc step after all steps complete, since unclamped intermediates are re-fed into the network and can compound.

## Related tools

- **RNAForecaster** (Provides predictCellFutures recursive prediction with built-in max-capping and negative-clamping of expression predictions) — https://github.com/rossinerbe/RNAForecaster.jl
- **Flux** (Machine-learning framework used to train the neural ODE whose recursive outputs are capped/clamped (version 0.13.0))
- **DiffEqFlux** (Neural ODE training framework (with Tsit5 solver) producing the network whose forecasts are filtered (version 1.46.0))

## Examples

```
using RNAForecaster; futureExpressionPreds = predictCellFutures(network, expressionData, 72)
```

## Evaluation signals

- No predicted gene expression value at any recursive step exceeds the allowed maximum (two times the maximum observed in training data, log space)
- No predicted value is negative after clamping at any step
- Median total counts per cell at 72 h remains at a realistic, non-exploding level consistent with the paper's Figure 6B (no runaway growth of predicted expression)
- For the scEU-seq run, the cap corresponds to a 1.2 log-unit increase over the training maximum
- Capped intermediate predictions, not raw ones, are what get fed back into the input nodes at each step

## Limitations

- The prior probability for gene expression outside the observed range is not learned by the network and is enforced post hoc; future work is needed to teach the network this prior instead
- The cap is a heuristic based on the training-data distribution; with only 405 training cells, rare cell-cycle events may still be poorly captured
- Filters like this control extreme predictions but do not guarantee the network can predict checkpoints or rare regulatory events

## Evidence
