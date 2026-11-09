---
name: variational-autoencoder-training
description: Use when single-cell expression matrices from paired time points (e.g.,
  BoolODE simulations or metabolic-labeling scRNA-seq) are available and you need
  to benchmark latent-embedding (VAE) inputs against by-gene neural ODE inputs for
  forecasting future expression states.
when_to_use_negative:
- When the goal is forecasting in the original gene-dimensional space — RNAForecaster's
  design goal is that 'it does not depend on a particular lower-dimensional embedding
  of the data but takes input in the gene-dimensional space'
- When data lack paired temporal measurements of the same cells, which the neural
  ODE training procedure requires
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: Flux.jl
  role: Machine-learning framework used to train the VAE and the neural ODE
- name: DiffEqFlux.jl
  role: Used with Flux.jl to train the neural ODE on the VAE-encoded input
- name: RNAForecaster
  role: Neural ODE forecasting package whose ensemble pipeline the VAE-input variant
    is benchmarked against
  repo: https://github.com/rossinerbe/RNAForecaster.jl
- name: BoolODE
  role: Generates the simulated single-cell expression time series used as training/testing
    data
  repo: https://github.com/FertigLab/RNAForecasterPaperCode
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2023.100793
    title: Transcriptomic forecasting with neural ordinary differential equations
  source_content_hash: sha256:1b845af0875984ba4d85e6d719d91e372aeb09553565800a3b0599ebee36d8fd
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
  - BoolODE
  - DiffEqFlux.jl
  - Flux.jl
  - RNAForecaster
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# variational-autoencoder-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a variational autoencoder (Flux.jl) to compress gene-space single-cell expression into a lower-dimensional latent representation, then feed that encoding through the RNAForecaster neural ODE to test whether dimensionality reduction improves or degrades multi-time-step transcriptomic forecasting.

## When to use

Use when single-cell expression matrices from paired time points (e.g., BoolODE simulations or metabolic-labeling scRNA-seq) are available and you need to benchmark latent-embedding (VAE) inputs against by-gene neural ODE inputs for forecasting future expression states.

## When NOT to use

- When the goal is forecasting in the original gene-dimensional space — RNAForecaster's design goal is that 'it does not depend on a particular lower-dimensional embedding of the data but takes input in the gene-dimensional space'
- When data lack paired temporal measurements of the same cells, which the neural ODE training procedure requires

## Inputs

- Single-cell expression count matrices from two paired time points (earlier counts as input, later counts as target), e.g., BoolODE simulated matrices from the RNAForecasterPaperCode repository
- Optionally, sparsity-filtered and log-normalized scRNA-seq matrices (e.g., scEU-seq rpeLabeling.h5ad)

## Outputs

- Trained VAE encoder/decoder mapping gene-space expression to a lower-dimensional latent space and back
- Per-time-point mean squared error time course of decoded recursive forecasts for the VAE-input neural ODE variant
- Comparison (significance test) of VAE-variant MSE versus the by-gene ensemble MSE at all simulated time points

## How to apply

Load the expression matrices (e.g., BoolODE simulations of random 10-gene networks with 801 time points and 2,000 cells per simulation, excluding stability-failing simulations to leave 111), and take the first two time points as the t=0/t=1 training pair. Train a VAE with Flux.jl to encode gene-space expression into a lower-dimensional latent space, then train the RNAForecaster neural ODE (Flux.jl/DiffEqFlux.jl) on the encoded input using the Tsit5 solver, default 1e-3 error tolerance, and 10 epochs, decoding predicted latent states back to gene space. Feed each decoded prediction recursively back through the VAE encoder and network input to forecast forward across all remaining time points. Compare mean squared error time courses against the baseline 10-network by-gene ensemble (median prediction across networks); the paper's result is that the VAE-input variant yields significantly higher MSE (p < 1e-7), with the largest degradation at early time points, because encoding discards information the gene-dimensional input preserves.

## Related tools

- **Flux.jl** (Machine-learning framework used to train the VAE and the neural ODE)
- **DiffEqFlux.jl** (Used with Flux.jl to train the neural ODE on the VAE-encoded input)
- **RNAForecaster** (Neural ODE forecasting package whose ensemble pipeline the VAE-input variant is benchmarked against) — https://github.com/rossinerbe/RNAForecaster.jl
- **BoolODE** (Generates the simulated single-cell expression time series used as training/testing data) — https://github.com/FertigLab/RNAForecasterPaperCode

## Examples

```
julia -e 'using Flux, DiffEqFlux, RNAForecaster; vae = trainVAE(expressionMatrix); latent = encodeData(expressionMatrix, vae)'  # install first with: ] add https://github.com/rossinerbe/RNAForecaster.jl
```

## Evaluation signals

- The VAE-input variant should show significantly higher forecast MSE than the 10-network by-gene ensemble at all time points (paper reports p < 1e-7, Figure S6)
- The largest MSE gap should occur at early time points, consistent with information loss from encoding
- The neural ODE should be trained with Tsit5, 1e-3 error tolerance, and 10 epochs, matching the RNAForecaster defaults
- Decoded predictions must remain within the allowed maximum (default 2x the max observed in training data, in log space)
- Recursive forecasts should remain stable across the full remaining time course (stability-check exclusions in the simulation set)

## Limitations

- Encoding into a lower-dimensional VAE latent space loses information and degrades prediction accuracy, especially at early time points
- The baseline ensemble approach requires increased computational resources, and the VAE variant adds encoding/decoding overhead
- RNAForecaster requires paired temporal measurements of the same cells, which standard scRNA-seq protocols cannot provide

## Evidence
