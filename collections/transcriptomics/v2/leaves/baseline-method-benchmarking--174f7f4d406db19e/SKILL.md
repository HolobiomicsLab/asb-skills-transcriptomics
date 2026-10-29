---
name: baseline-method-benchmarking
description: Use when a new trajectory/pseudotime method must be validated against
  published baselines on shared datasets, especially to quantify robustness under
  random cell subsampling (e.g. 90% ME subsampling over 50 repeats).
when_to_use_negative:
- The analysis goal is trajectory inference or branch detection itself, not fair comparison
  of methods on identical inputs
- The dataset has no reference full-data ordering available, so subsample-to-full
  Spearman correlations cannot be computed
- Only one method is of interest and no baselines (Scanpy, Monocle2) can be run on
  the same preprocessed data
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: DTFLOW
  role: Method under test; produces pseudotime orderings via BKFD dimension reduction
    and random walk with restart (k=10, p=0.9)
  repo: https://github.com/statway/DTFLOW
- name: Scanpy
  role: Baseline pseudotime method; also used for pre-processing (removal of 10 corrupted
    gene names in MMP)
- name: Monocle2
  role: Baseline pseudotime method compared against DTFLOW on ME and MMP orderings
- name: Python
  role: Implementation language for running DTFLOW and computing Spearman correlation
    statistics
  repo: https://github.com/statway/DTFLOW
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:d014add86e5d74c230d81e3464f4c5ceaf5a5c80af16d9da9374e48f4e1c6fc5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Monocle2
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# baseline-method-benchmarking

## Summary

Benchmark a proposed pseudotime method (DTFLOW) against established baselines (Scanpy, Monocle2) by running all methods on the same datasets and on repeated random cell subsamples, then comparing Spearman/Kendall rank correlations of the orderings against the full-data reference.

## When to use

Use when a new trajectory/pseudotime method must be validated against published baselines on shared datasets, especially to quantify robustness under random cell subsampling (e.g. 90% ME subsampling over 50 repeats).

## When NOT to use

- The analysis goal is trajectory inference or branch detection itself, not fair comparison of methods on identical inputs
- The dataset has no reference full-data ordering available, so subsample-to-full Spearman correlations cannot be computed
- Only one method is of interest and no baselines (Scanpy, Monocle2) can be run on the same preprocessed data

## Inputs

- ME high-throughput RT-PCR mouse embryo dataset (48 genes, 438 cells)
- MMP mouse myeloid progenitor MARS-seq dataset (3451 genes, 2730 cells; 10 genes with corrupted names removed via Scanpy pre-processing)
- Pseudotime orderings from DTFLOW (k=10, p=0.9), Scanpy, and Monocle2 on full and subsampled data

## Outputs

- Per-subsample Spearman rank correlations between subsample and full-data orderings for each method-dataset pair
- Aggregated mean, standard deviation, and variance statistics per method-dataset pair written to robustness_correlations.csv

## How to apply

Run all competing methods on identical inputs: DTFLOW with k=10 nearest neighbors and restart probability p=0.9, Scanpy, and Monocle2 on the full ME dataset (48 genes, 438 cells) and MMP MARS-seq dataset (3451 genes, 2730 cells after removing 10 genes with corrupted names via Scanpy pre-processing). For robustness, randomly sample 90% of ME cells (394 of 438) 50 times and rerun every method per subsample; for MMP, subsample 2500 of 2730 cells. Compute Spearman rank correlation between each subsample ordering and the corresponding full-data ordering, then aggregate mean, standard deviation, and variance per method-dataset pair (Kendall coefficients, e.g. 0.862/0.796/0.761 for DTFLOW/Scanpy/Monocle2 on ME, serve as accuracy references). Write the aggregated statistics to a summary CSV (robustness_correlations.csv). Rationale: a method that is truly superior must match baselines on accuracy while showing higher mean correlation and lower variance across subsamples.

## Related tools

- **DTFLOW** (Method under test; produces pseudotime orderings via BKFD dimension reduction and random walk with restart (k=10, p=0.9)) — https://github.com/statway/DTFLOW
- **Scanpy** (Baseline pseudotime method; also used for pre-processing (removal of 10 corrupted gene names in MMP))
- **Monocle2** (Baseline pseudotime method compared against DTFLOW on ME and MMP orderings)
- **Python** (Implementation language for running DTFLOW and computing Spearman correlation statistics) — https://github.com/statway/DTFLOW

## Examples

```
from DTFLOW import *; z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9); dftimes = z.get_dtflow_times(root_list)
```

## Evaluation signals

- robustness_correlations.csv exists and contains mean, standard deviation, and variance fields for every method-dataset pair
- DTFLOW shows a higher mean Spearman correlation and smaller variance than Scanpy and Monocle2 across the 50 ME subsamples
- DTFLOW Kendall coefficients on ME (0.862) and MFG (0.761) exceed Scanpy (0.796, 0.702) and Monocle2 (0.761, 0.569), matching the paper's reported values
- ME subsamples contain exactly 394 of 438 cells per draw and the MMP subsample contains exactly 2500 of 2730 cells
- Each method is run on the identically preprocessed data (same gene set, e.g. MMP with 3451 genes after corrupted-name removal) so the comparison is fair

## Limitations

- Random 90% subsampling results are stochastic; the paper aggregates over 50 repeats but individual runs may vary
- DTFLOW parameters (Gaussian kernel, number of neighbors k, restart probability 1-p) may need to vary from dataset to dataset and are not standardized
- BKFD uses the same restart probability for all nodes, which may limit the effectiveness of the random walk
- Monocle2's lower robustness may partly reflect sensitivity to subsampling rather than ordering accuracy per se (Kendall coefficient gap with DTFLOW on ME is small: 0.862 vs 0.761)

## Evidence

- [results] Under the 50-times-repeated 90% subsampling protocol on the ME dataset, DTFLOW and Scanpy show better robustness than Monocle2, and DTFLOW's variance of Spearman correlation coefficients is smaller than Scanpy's, making it the most stable method.: [evidence span withheld — anchor status: not_found]
- [abstract] Simulation results suggest that DTFLOW has superior accuracy and strong robustness properties for constructing pseudotime trajectories.: "Simulation results suggest that DTFLOW has superior accuracy and strong robustness properties for constructing pseudotime trajectories."
- [results] Kendall rank correlation coefficients on ME for DTFLOW, Scanpy, Monocle2.: "The calculated Kendall rank correlation coefficients are 0.862, 0.796, 0.761 for DTFLOW, Scanpy, and Monocle2, respectively"
- [methods] 10 genes with corrupted names are removed from our analysis based on the pre-processing of Scanpy.: "10 genes with corrupted names are removed from our analysis based on the pre-processing of Scanpy."
- [results] DTFLOW parameters: k=10 neighbors, restart probability p=0.9.: "We test different values of k and find that the results are better if k=10"
