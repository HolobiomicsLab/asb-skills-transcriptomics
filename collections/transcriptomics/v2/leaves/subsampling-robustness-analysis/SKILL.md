---
name: subsampling-robustness-analysis
description: Use when comparing the robustness of pseudotime inference methods (e.g.,
  DTFLOW, Scanpy, Monocle2) on a single-cell dataset where orderings from random cell
  subsamples should be checked against the full-data ordering.
when_to_use_negative:
- The goal is accuracy against known time stages (e.g., Kendall rank correlation against
  developmental stages), not subsampling stability — use the paper's accuracy protocol
  instead
- The dataset has so few cells that removing 10% of cells would eliminate rare developmental
  states or branches
- Only a single fixed train/test split is needed rather than repeated random subsampling
  (e.g., the one-off 2500-cell MMP subsample without repetition)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: DTFLOW
  role: Primary pseudotime method under test; run with kNN k=10 and restart probability
    p=0.9 on each cell subsample to obtain orderings whose stability is measured
  repo: https://github.com/statway/DTFLOW
- name: Scanpy
  role: Baseline pseudotime method and pre-processing tool (removal of 10 MMP genes
    with corrupted names) whose subsample orderings are compared for robustness
- name: Monocle2
  role: Baseline pseudotime method whose subsample orderings are compared for robustness
    against DTFLOW and Scanpy
- name: Python
  role: Implementation language for the subsampling loop, Spearman correlation computation,
    and CSV summary statistics
  repo: https://github.com/statway/DTFLOW
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:bd867d2e18aa9339097456f304b1a14175476e02201cc1ad59c1d65e875d419c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Monocle2
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# subsampling-robustness-analysis

## Summary

Assesses pseudotime-ordering stability of DTFLOW, Scanpy, and Monocle2 by repeatedly subsampling single-cell datasets (90% of ME cells 50 times; 2500 of 2730 MMP cells) and computing Spearman rank correlations of subsample orderings against full-data orderings, aggregated into mean, standard deviation, and variance.

## When to use

Use when comparing the robustness of pseudotime inference methods (e.g., DTFLOW, Scanpy, Monocle2) on a single-cell dataset where orderings from random cell subsamples should be checked against the full-data ordering.

## When NOT to use

- The goal is accuracy against known time stages (e.g., Kendall rank correlation against developmental stages), not subsampling stability — use the paper's accuracy protocol instead
- The dataset has so few cells that removing 10% of cells would eliminate rare developmental states or branches
- Only a single fixed train/test split is needed rather than repeated random subsampling (e.g., the one-off 2500-cell MMP subsample without repetition)

## Inputs

- ME RT-PCR mouse embryo dataset (48 genes, 438 cells) as a N cells x D genes expression matrix (raw_df with time_stages)
- MMP MARS-seq mouse myeloid progenitor dataset (3451 genes, 2730 cells; 10 genes with corrupted names removed via Scanpy pre-processing)
- Full-data reference pseudotime orderings from DTFLOW, Scanpy, and Monocle2

## Outputs

- Per-subsample Spearman rank correlation coefficients for each method-dataset pair
- robustness_correlations.csv containing mean, standard deviation, and variance of Spearman correlations per method-dataset pair

## How to apply

Load the target dataset — e.g., the ME RT-PCR mouse embryo dataset (48 genes, 438 cells) or the MMP MARS-seq dataset (3451 genes, 2730 cells after removal of 10 genes with corrupted names via Scanpy pre-processing). Randomly sample 90% of the ME cells (394 of 438) 50 times, and subsample 2500 of the 2730 MMP cells. On each subsample, run DTFLOW (kNN k=10, restart probability p=0.9), Scanpy, and Monocle2 to obtain pseudotime orderings, then compute the Spearman rank correlation of each subsample ordering against the corresponding full-data ordering. Aggregate correlations per method-dataset pair into mean, standard deviation, and variance, and write the summary statistics table to robustness_correlations.csv. DTFLOW with k=10 and p=0.9 is expected to show high mean Spearman correlations with the smallest variance across subsamples, as reported in the paper.

## Related tools

- **DTFLOW** (Primary pseudotime method under test; run with kNN k=10 and restart probability p=0.9 on each cell subsample to obtain orderings whose stability is measured) — https://github.com/statway/DTFLOW
- **Scanpy** (Baseline pseudotime method and pre-processing tool (removal of 10 MMP genes with corrupted names) whose subsample orderings are compared for robustness)
- **Monocle2** (Baseline pseudotime method whose subsample orderings are compared for robustness against DTFLOW and Scanpy)
- **Python** (Implementation language for the subsampling loop, Spearman correlation computation, and CSV summary statistics) — https://github.com/statway/DTFLOW

## Examples

```
M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9); dftimes = z.get_dtflow_times(root_list)
```

## Evaluation signals

- robustness_correlations.csv exists and contains mean, standard deviation, and variance fields for every method-dataset pair
- DTFLOW robustness statistics match or approach the paper's reported values: high mean Spearman correlations with small variance across the 50 ME subsamples
- DTFLOW's variance of Spearman correlation coefficients on the ME dataset is smaller than Scanpy's, and both DTFLOW and Scanpy outperform Monocle2 in robustness
- Each subsample contains the correct number of cells (394 of 438 ME cells per draw; 2500 of 2730 MMP cells) and 50 independent random draws were performed for the ME dataset

## Limitations

- Parameter values (Gaussian kernel, number of closest neighbors k, restart probability 1-p) may vary from dataset to dataset and are not standardized
- The one-off 2500-cell MMP subsample does not follow the 50-times-repeated protocol used for ME, limiting direct statistical comparison across datasets
- MMP results depend on Scanpy pre-processing to remove 10 genes with corrupted names; skipping this step changes the input data

## Evidence

- [methods] The second dataset is the mouse myeloid progenitor (MMP) MARS-seq dataset that contains 2730 single cells and 3451 informative genes: "The second dataset is the mouse myeloid progenitor (MMP) MARS-seq dataset that contains 2730 single cells and 3451 informative genes"
- [methods] 10 genes with corrupted names are removed from our analysis based on the pre-processing of Scanpy.: "10 genes with corrupted names are removed from our analysis based on the pre-processing of Scanpy."
- [results] We test different values of k and find that the results are better if k=10: "We test different values of k and find that the results are better if k=10"
- [results] the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work: "the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work"
- [results] the ME single-cell dataset with 48 genes and 438 single cells: "the ME single-cell dataset with 48 genes and 438 single cells"
- [abstract] Simulation results suggest that DTFLOW has superior accuracy and strong robustness properties for constructing pseudotime trajectories.: "Simulation results suggest that DTFLOW has superior accuracy and strong robustness properties for constructing pseudotime trajectories."
- [readme] S = z.get_diffusionRWR(p): "Here 1-p is the restart probability of RWR processes."
