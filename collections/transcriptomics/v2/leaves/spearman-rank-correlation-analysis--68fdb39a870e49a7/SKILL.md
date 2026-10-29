---
name: spearman-rank-correlation-analysis
description: Use when comparing pseudotime (or any ranked) orderings between a full
  single-cell dataset and repeated random cell subsamples, to measure ordering robustness
  of trajectory-inference methods such as DTFLOW, Scanpy, or Monocle2.
when_to_use_negative:
- Input is raw gene expression or a single pseudotime ordering only — Spearman correlation
  requires two ranked orderings over the same cells to compare
- The goal is branch/topology detection or accuracy against known time stages — the
  paper uses Kendall rank correlation against time-stage labels for accuracy, not
  this subsampling robustness protocol
- Datasets other than the ME/MMP settings without a defined full-data reference ordering
  to correlate subsample orderings against
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DTFLOW
  role: Trajectory method whose pseudotime orderings are compared across 90% random
    subsamples (ME, 50 repeats; kNN k=10, restart probability p=0.9)
  repo: https://github.com/statway/DTFLOW
- name: Scanpy
  role: Baseline pseudotime method and pre-processing step (removal of 10 genes with
    corrupted names from the MMP dataset) before subsample correlations are computed
- name: Monocle2
  role: Baseline pseudotime method run on full data and each subsample to obtain orderings
    correlated against full-data orderings
- name: Python
  role: Implementation language for the subsampling loop, Spearman correlation computation,
    and aggregation into robustness_correlations.csv
  repo: https://github.com/statway/DTFLOW
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:88a6db2ca93a83b66212aa0e6b0bad144bb6318652de792e83547010fab05474
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Monocle2
  - Python
  - Scanpy
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spearman-rank-correlation-analysis

## Summary

Quantifies the stability of pseudotime orderings by computing Spearman rank correlations between cell orderings obtained on randomly subsampled data (90% of ME cells, 2500 of 2730 MMP cells) and the corresponding full-data orderings, aggregating mean, standard deviation, and variance per method. In the DTFLOW paper this robustness protocol distinguishes DTFLOW from Scanpy and Monocle2.

## When to use

Use when comparing pseudotime (or any ranked) orderings between a full single-cell dataset and repeated random cell subsamples, to measure ordering robustness of trajectory-inference methods such as DTFLOW, Scanpy, or Monocle2.

## When NOT to use

- Input is raw gene expression or a single pseudotime ordering only — Spearman correlation requires two ranked orderings over the same cells to compare
- The goal is branch/topology detection or accuracy against known time stages — the paper uses Kendall rank correlation against time-stage labels for accuracy, not this subsampling robustness protocol
- Datasets other than the ME/MMP settings without a defined full-data reference ordering to correlate subsample orderings against

## Inputs

- ME high-throughput RT-PCR mouse embryo dataset (48 genes, 438 cells) with full-data pseudotime ordering
- MMP mouse myeloid progenitor MARS-seq dataset (3451 genes, 2730 cells; 10 genes with corrupted names removed via Scanpy pre-processing)
- Pseudotime orderings per cell from DTFLOW (k=10, p=0.9), Scanpy, and Monocle2 on full data and on 50 repeated 90% ME subsamples / 2500-cell MMP subsample

## Outputs

- Per-subsample Spearman rank correlation coefficients between each subsample ordering and the full-data ordering, per method and dataset
- Aggregated mean, standard deviation, and variance statistics per method-dataset pair written to robustness_correlations.csv

## How to apply

Load the ME RT-PCR mouse embryo dataset (48 genes, 438 cells) and the MMP MARS-seq dataset (3451 genes, 2730 cells after removal of 10 genes with corrupted names via Scanpy pre-processing). For the ME dataset, randomly sample 90% of the cells (394 of 438) 50 times; for MMP, subsample 2500 of the 2730 cells. Run each method (DTFLOW with kNN k=10 and restart probability p=0.9, Scanpy, and Monocle2) on every subsample and on the full data to obtain pseudotime orderings. Compute the Spearman rank correlation between each subsample ordering and the corresponding full-data ordering, then aggregate per method and dataset into mean, standard deviation, and variance, writing the summary to robustness_correlations.csv. A more stable method shows a high mean correlation with small variance; in the paper DTFLOW's variance of correlations is smaller than Scanpy's, and both DTFLOW and Scanpy are more robust than Monocle2.

## Related tools

- **DTFLOW** (Trajectory method whose pseudotime orderings are compared across 90% random subsamples (ME, 50 repeats; kNN k=10, restart probability p=0.9)) — https://github.com/statway/DTFLOW
- **Scanpy** (Baseline pseudotime method and pre-processing step (removal of 10 genes with corrupted names from the MMP dataset) before subsample correlations are computed)
- **Monocle2** (Baseline pseudotime method run on full data and each subsample to obtain orderings correlated against full-data orderings)
- **Python** (Implementation language for the subsampling loop, Spearman correlation computation, and aggregation into robustness_correlations.csv) — https://github.com/statway/DTFLOW

## Examples

```
from DTFLOW import *; z = DTFLOW(raw_df,time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9); dftimes = z.get_dtflow_times(root_list)
```

## Evaluation signals

- robustness_correlations.csv exists and contains mean, standard deviation, and variance fields for every method-dataset pair (DTFLOW/Scanpy/Monocle2 x ME/MMP)
- ME subsample size is exactly 394 cells (90% of 438) and each of the 50 repeats yields one Spearman coefficient per method
- MMP correlations are computed against a 2500-cell subsample of the 2730-cell dataset (which excludes 10 genes with corrupted names removed by Scanpy pre-processing)
- DTFLOW shows high mean Spearman correlation with variance smaller than Scanpy's, approaching the paper's reported strong-robustness values
- Ranking of robustness matches the paper: DTFLOW and Scanpy more robust than Monocle2 on the ME dataset

## Limitations

- Parameter values (Gaussian kernel, number of closest neighbors k, restart probability) may vary from dataset to dataset and are not standardized
- Robustness statistics depend on the specific random subsampling protocol (90%, 50 repeats for ME; single 2500-cell subsample for MMP), so results are protocol-specific
- The protocol measures ordering stability, not accuracy against true time stages

## Evidence

- [results] p=0.9 restart probability: "the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work"
- [methods] MMP dataset dimensions: "The second dataset is the mouse myeloid progenitor (MMP) MARS-seq dataset that contains 2730 single cells and 3451 informative genes"
- [methods] Corrupted-gene removal: "10 genes with corrupted names are removed from our analysis based on the pre-processing of Scanpy."
- [readme] RWR step in DTFLOW pipeline: "Here 1-p is the restart probability of RWR processes."
