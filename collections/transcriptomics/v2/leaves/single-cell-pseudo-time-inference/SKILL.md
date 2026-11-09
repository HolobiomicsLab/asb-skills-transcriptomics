---
name: single-cell-pseudo-time-inference
description: Use when a single-cell RNA-seq count matrix captures cells from ordered
  developmental stages and the goal is to infer cell ordering (pseudo-time) and quantify
  it against known capture/stage time.
when_to_use_negative:
- The dataset lacks any ordered biological process (stages/capture times) against
  which a trajectory can be meaningfully evaluated
- Only a single trajectory-inference result is needed without low-dimensional embedding
  and scDHA's clustering/classification pipeline
- Input is already a low-dimensional embedding from another framework where scDHA's
  noise-detachment autoencoder step is not part of the validated workflow
tools:
- name: scDHA
  role: 'Main framework: non-negative kernel autoencoder + stacked Bayesian self-learning
    module produce the low-dimensional embedding, and scDHA.pt infers pseudo-time
    from that embedding'
  repo: github.com/duct317/scDHA
- name: Monocle
  role: State-of-the-art trajectory inference baseline run on the same preprocessed
    datasets for R-squared benchmarking
  repo: cole-trapnell-lab/monocle3
- name: TSCAN
  role: Trajectory inference baseline used for pseudo-time comparison on Yan, Goolam,
    and Deng
- name: Slingshot
  role: Trajectory inference baseline used for pseudo-time comparison on the three
    mouse embryo datasets
- name: SCANPY
  role: Trajectory inference baseline (considered state-of-the-art alongside Monocle
    and TSCAN in the paper) used for R-squared comparison
- name: CIDR
  role: Mentioned among widely-used scRNA-seq dimensionality-reduction/clustering
    tools in the article's comparison landscape; not part of the pseudo-time benchmark
    itself
  repo: VCCRI/CIDR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:955a4b942d46d7c48f3bf7ff891c3440b8af60e462dbced2167baa0ee07ac408
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CIDR
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-pseudo-time-inference

## Summary

Reconstruct ordered developmental trajectories from scRNA-seq data by embedding cells in scDHA's low-dimensional space (non-negative kernel autoencoder + stacked Bayesian self-learning module) and applying scDHA.pt, benchmarking R-squared against Monocle, TSCAN, Slingshot, and SCANPY.

## When to use

Use when a single-cell RNA-seq count matrix captures cells from ordered developmental stages and the goal is to infer cell ordering (pseudo-time) and quantify it against known capture/stage time.

## When NOT to use

- The dataset lacks any ordered biological process (stages/capture times) against which a trajectory can be meaningfully evaluated
- Only a single trajectory-inference result is needed without low-dimensional embedding and scDHA's clustering/classification pipeline
- Input is already a low-dimensional embedding from another framework where scDHA's noise-detachment autoencoder step is not part of the validated workflow

## Inputs

- Single-cell RNA-seq count matrix (rows = samples/cells, columns = genes), log2-transformed
- Known embryonic capture/stage time labels per cell (for R-squared evaluation)
- scDHA dimension reduction output (low-dimensional cell embedding)

## Outputs

- Pseudo-time ordering of cells from scDHA.pt
- R-squared values comparing scDHA.pt vs Monocle, TSCAN, Slingshot, SCANPY per dataset
- Comparison table saved as pseudo_time_r2_results.csv

## How to apply

Log-transform the count matrix (log2(counts + 1), rows as samples, columns as genes) and run scDHA's two-module pipeline: a non-negative kernel autoencoder that removes genes or components with insignificant contributions (judged by encoder weight distribution), followed by the stacked Bayesian self-learning network (VAE-based) that projects data to a compact low-dimensional space. Feed that dimension reduction output into scDHA.pt to generate pseudo-time; set seed (e.g. seed = 1) for reproducibility. For benchmarking, run Monocle, TSCAN, Slingshot, and SCANPY on the same preprocessed data, compute R-squared between each inferred pseudo-time and known embryonic capture/stage time, and assemble a comparison table (scDHA.pt should reach R-squared 0.93 vs Monocle 0.84 on Yan, ranking highest on Yan, Goolam, and Deng). The rationale is that scDHA detaches technical noise/dropout from biological signal before trajectory inference, improving ordering accuracy and keeping complexity near-linear.

## Related tools

- **scDHA** (Main framework: non-negative kernel autoencoder + stacked Bayesian self-learning module produce the low-dimensional embedding, and scDHA.pt infers pseudo-time from that embedding) — github.com/duct317/scDHA
- **Monocle** (State-of-the-art trajectory inference baseline run on the same preprocessed datasets for R-squared benchmarking) — cole-trapnell-lab/monocle3
- **TSCAN** (Trajectory inference baseline used for pseudo-time comparison on Yan, Goolam, and Deng)
- **Slingshot** (Trajectory inference baseline used for pseudo-time comparison on the three mouse embryo datasets)
- **SCANPY** (Trajectory inference baseline (considered state-of-the-art alongside Monocle and TSCAN in the paper) used for R-squared comparison)
- **CIDR** (Mentioned among widely-used scRNA-seq dimensionality-reduction/clustering tools in the article's comparison landscape; not part of the pseudo-time benchmark itself) — VCCRI/CIDR

## Examples

```
library(scDHA); data('Goolam'); data <- log2(t(Goolam$data) + 1); emb <- scDHA(data, seed = 1); pt <- scDHA.pt(emb)
```

## Evaluation signals

- R-squared between inferred pseudo-time and known embryonic capture time must match the reported values on Yan (scDHA.pt 0.93 vs Monocle 0.84)
- scDHA.pt must rank highest among the five tools (scDHA.pt, Monocle, TSCAN, Slingshot, SCANPY) on each of the three datasets
- Reproduced comparison table matches the paper-reported R-squared values when saved as pseudo_time_r2_results.csv
- Setting the same seed (e.g. seed = 1) reproduces the clustering and pseudo-time result
- scDHA clustering on the Goolam example yields an adjustedRandIndex (mclust) consistent with the paper's clustering accuracy before trajectory inference

## Limitations

- scDHA's accuracy claims for the autoencoder modules are partly stated as expectations rather than demonstrated in the main discussion
- The scVI comparative analysis is deferred to supplementary materials (Supplementary Section 6, Figs. 25–32) rather than the main text
- The scDHA machine learning framework changed from Tensorflow to Torch; Python environment is no longer required, but libtorch must be installed (torch::install_torch())
- Discussion claims flexibility to other domains (cancer, obesity, aging) are not supported by experiments within the paper

## Evidence

- [intro] Among a number of trajectory inference tools, Monocle25, TSCAN26, Slingshot27, and SCANPY13 are considered state-of-the-art: "Among a number of trajectory inference tools, Monocle25, TSCAN26, Slingshot27, and SCANPY13 are considered state-of-the-art"
- [readme] scDHA.pt: generating pseudotime. The input is demension reduction output.: [evidence span withheld — anchor status: external_doc]
- [intro] Based on the weight distribution of the encoder, scDHA removes genes or components that have insignificant contributions to the representation: "Based on the weight distribution of the encoder, scDHA removes genes or components that have insignificant contributions to the representation"
- [readme] The result is reproducible by setting seed for these functions.: [evidence span withheld — anchor status: external_doc]
