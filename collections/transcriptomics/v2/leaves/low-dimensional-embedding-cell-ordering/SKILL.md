---
name: low-dimensional-embedding-cell-ordering
description: Use when scRNA-seq cells (e.g. mouse embryo datasets Yan, Goolam, Deng)
  already have a scDHA low-dimensional representation and the goal is a developmental
  pseudo-time ordering whose accuracy can be scored as R-squared against known capture/stage
  time.
when_to_use_negative:
- The data have no scDHA low-dimensional embedding — scDHA.pt's input is the dimension
  reduction output, so raw counts must first be embedded via scDHA
- 'The analysis stack is scVI alone: ''The scVI package itself is not capable of clustering,
  visualization, classification, and pseudo-time inference'''
- There is no reference time/stage information and the goal is only qualitative trajectory
  visualization rather than an R-squared-scored ordering
tools:
- name: scDHA
  role: Embeds cells into a low-dimensional space (non-negative kernel autoencoder
    + stacked Bayesian self-learning module) and infers pseudo-time via scDHA.pt on
    that embedding
  repo: github.com/duct317/scDHA
- name: Monocle
  role: State-of-the-art trajectory inference baseline producing a competing pseudo-time
    ordering for R-squared comparison
  repo: cole-trapnell-lab/monocle3
- name: TSCAN
  role: State-of-the-art trajectory inference baseline producing a competing pseudo-time
    ordering for R-squared comparison
- name: Slingshot
  role: State-of-the-art trajectory inference baseline producing a competing pseudo-time
    ordering for R-squared comparison
- name: SCANPY
  role: State-of-the-art trajectory inference baseline producing a competing pseudo-time
    ordering for R-squared comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:0b18fcdcaa3fd2b6fdfa882208119f855dc769c2c7720e8bd5de1cd82b4cf2ae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle
  - SCANPY
  - Slingshot
  - TSCAN
  - scDHA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# low-dimensional-embedding-cell-ordering

## Summary

Infers single-cell pseudo-time by ordering cells along a trajectory in a scDHA low-dimensional embedding (via scDHA.pt), and benchmarks the ordering against state-of-the-art trajectory tools (Monocle, TSCAN, Slingshot, SCANPY) using R-squared against known embryonic capture stages.

## When to use

Use when scRNA-seq cells (e.g. mouse embryo datasets Yan, Goolam, Deng) already have a scDHA low-dimensional representation and the goal is a developmental pseudo-time ordering whose accuracy can be scored as R-squared against known capture/stage time.

## When NOT to use

- The data have no scDHA low-dimensional embedding — scDHA.pt's input is the dimension reduction output, so raw counts must first be embedded via scDHA
- The analysis stack is scVI alone: 'The scVI package itself is not capable of clustering, visualization, classification, and pseudo-time inference'
- There is no reference time/stage information and the goal is only qualitative trajectory visualization rather than an R-squared-scored ordering

## Inputs

- scRNA-seq count matrix (rows = samples/cells, columns = genes), e.g. GSE36552 (Yan), E-MTAB-3321 (Goolam), GSE45719 (Deng)
- Known embryonic capture/stage time labels per cell for validation
- scDHA low-dimensional embedding output (input to scDHA.pt)

## Outputs

- Per-cell pseudo-time ordering from scDHA.pt on the scDHA embedding
- R-squared values between each tool's pseudo-time and known capture time
- Comparison table pseudo_time_r2_results.csv (scDHA.pt vs Monocle, TSCAN, Slingshot, SCANPY)

## How to apply

Download the embryo scRNA-seq count matrices (GSE36552/Yan, E-MTAB-3321/Goolam, GSE45719/Deng), log2-transform counts (log2(data + 1)) with samples as rows and genes as columns, and embed cells with the scDHA package — a non-negative kernel autoencoder that removes genes/components with insignificant encoder-weight contributions, followed by the stacked Bayesian self-learning module that projects data onto a low-dimensional space with repeated perturbation for generalization. Apply scDHA.pt on this embedding output to generate pseudo-time, setting a seed for reproducibility. Run Monocle, TSCAN, Slingshot, and SCANPY on the same preprocessed data as competing orderings. Compute R-squared between each tool's inferred pseudo-time and the known embryonic capture time; on Yan the expected result is scDHA.pt at 0.93 versus Monocle's 0.84, with scDHA.pt highest on every dataset.

## Related tools

- **scDHA** (Embeds cells into a low-dimensional space (non-negative kernel autoencoder + stacked Bayesian self-learning module) and infers pseudo-time via scDHA.pt on that embedding) — github.com/duct317/scDHA
- **Monocle** (State-of-the-art trajectory inference baseline producing a competing pseudo-time ordering for R-squared comparison) — cole-trapnell-lab/monocle3
- **TSCAN** (State-of-the-art trajectory inference baseline producing a competing pseudo-time ordering for R-squared comparison)
- **Slingshot** (State-of-the-art trajectory inference baseline producing a competing pseudo-time ordering for R-squared comparison)
- **SCANPY** (State-of-the-art trajectory inference baseline producing a competing pseudo-time ordering for R-squared comparison)

## Examples

```
data('Goolam'); data <- t(Goolam$data); data <- log2(data + 1); result <- scDHA(data, seed = 1); pt <- scDHA.pt(result, seed = 1)
```

## Evaluation signals

- On the Yan dataset, reproduced R-squared must show scDHA.pt at 0.93 versus Monocle's 0.84
- scDHA.pt must rank highest in R-squared among the five tools (scDHA.pt, Monocle, TSCAN, Slingshot, SCANPY) on each of Yan, Goolam, and Deng
- The assembled pseudo_time_r2_results.csv must match the paper-reported R-squared values for every tool/dataset pair
- Results must be reproducible: scDHA/scDHA.pt outputs are reproducible by setting a seed (e.g. seed = 1)
- Input matrix orientation check: rows = samples, columns = genes, log2-transformed counts (log2(data + 1))

## Limitations

- The article's discussion section acknowledges no explicit limitations or caveats; accuracy claims there are stated as expectations rather than demonstrated results
- scDHA.pt requires the scDHA dimension reduction output as input — it cannot be applied directly to raw count matrices
- Validation depends on known capture/stage time labels; datasets without such labels cannot be scored by R-squared

## Evidence

- [intro] state-of-the-art trajectory baselines: "Among a number of trajectory inference tools, Monocle25, TSCAN26, Slingshot27, and SCANPY13 are considered state-of-the-art"
- [intro] scDHA scope includes pseudo-time inference: "the scDHA software package conducts cell segregation through unsupervised learning, dimension reduction and visualization, cell classification, and time-trajectory inference"
- [readme] scDHA.pt input contract: [evidence span withheld — anchor status: external_doc]
- [readme] scDHA main function input format: [evidence span withheld — anchor status: external_doc]
- [readme] reproducibility via seed: [evidence span withheld — anchor status: external_doc]
