---
name: single-cell-rna-seq-simulation
description: Use when a benchmark with known ground-truth cell types and latent progression
  is needed to compare unsupervised feature selection or trajectory inference methods
  on simulated scRNA-seq data.
when_to_use_negative:
- The goal is to benchmark on real experimental data where no ground-truth latent
  progression or cell-type labels exist (use real scRNA-seq with validated annotations
  instead).
- You need to simulate non-transcriptomic modalities (e.g., iterative immunofluorescence
  protein imaging); SymSim parameters here model UMI-protocol scRNA-seq only.
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: SymSim
  role: R package (v0.0.0.9000) used to simulate tree differentiation trajectories
    with configurable depth, capture efficiency, and evolutionary functional factors
    as ground-truth benchmark data
  repo: https://github.com/YosefLab/SymSim
- name: delve_benchmark
  role: Python package that runs the feature selection and evaluation pipeline on
    the simulated datasets via the fs class (select() and evaluate_select())
  repo: https://github.com/jranek/delve_benchmark
- name: Python
  role: Used for PCA preprocessing (n_pcs = 50), kNN classification evaluation, and
    Kendall rank correlation between diffusion pseudotime and ground-truth latent
    progression
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:8b9cad09db98032e9208b683fbaa7696b2f35e0abfb620d17bc67badab26eb4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - Python
  - R
  - SymSim
  - delve_benchmark
  - numpy
  - pandas
  - scikit-learn (sklearn)
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- single-cell-rnaseq-simulation
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-rna-seq-simulation

## Summary

Generate synthetic single-cell RNA-seq datasets with SymSim (tree differentiation trajectories, configurable cell counts, genes, capture efficiency) to create ground-truth benchmarks for evaluating feature selection and trajectory-preservation methods.

## When to use

Use when a benchmark with known ground-truth cell types and latent progression is needed to compare unsupervised feature selection or trajectory inference methods on simulated scRNA-seq data.

## When NOT to use

- The goal is to benchmark on real experimental data where no ground-truth latent progression or cell-type labels exist (use real scRNA-seq with validated annotations instead).
- You need to simulate non-transcriptomic modalities (e.g., iterative immunofluorescence protein imaging); SymSim parameters here model UMI-protocol scRNA-seq only.

## Inputs

- SymSim simulation parameters (n cells, n genes, n cell types, sigma, alpha_mean, alpha_sd, evf_type, protocol, depth_mean, depth_sd)
- Simulated count matrices with ground-truth cell-type labels and latent progression values
- PCA embedding (n_pcs = 50) of each simulated dataset

## Outputs

- Simulated SymSim tree-trajectory datasets (10,000 cells x 20,000 genes, 4 cell types) with ground truth
- Feature-selection benchmark table (symsim_feature_selection_results.csv) with median classification accuracy and pseudotime Kendall correlation per method x alpha_mean x p
- Per-method ranked feature lists at p = 2000, 1000, 500, 250, 100, 50

## How to apply

Simulate differentiation trajectories with SymSim v0.0.0.9000 in R (e.g., five tree trajectories with 10,000 cells, 20,000 genes, 4 cell types, evf_type = continuous, protocol = UMI, depth_mean = 5e5, depth_sd = 3e4, varying alpha_mean like (0.01, 0.02, 0.03, 0.04, 0.05) with alpha_sd = 0.02 and reduced mRNA capture efficiency sigma = 0.6/0.4). Preprocess each simulated count matrix in Python, e.g. reduce to 50 principal components, to mirror the format the downstream feature selection methods expect. Run the methods under test (DELVE, Laplacian Score, Hotspot, MCFS, SCMER, neighborhood variance, max variance, HVG, random) on the PCA embedding, selecting the top p features (e.g., 2000 down to 50). Evaluate with k-nearest neighbor cell-type classification (k = 3, 3-fold stratified cross-validation) and Kendall rank correlation between diffusion pseudotime and the ground-truth latent progression, aggregating medians across datasets and writing results to a CSV such as symsim_feature_selection_results.csv.

## Related tools

- **SymSim** (R package (v0.0.0.9000) used to simulate tree differentiation trajectories with configurable depth, capture efficiency, and evolutionary functional factors as ground-truth benchmark data) — https://github.com/YosefLab/SymSim
- **delve_benchmark** (Python package that runs the feature selection and evaluation pipeline on the simulated datasets via the fs class (select() and evaluate_select())) — https://github.com/jranek/delve_benchmark
- **Python** (Used for PCA preprocessing (n_pcs = 50), kNN classification evaluation, and Kendall rank correlation between diffusion pseudotime and ground-truth latent progression)

## Examples

```
library(devtools); devtools::install_github("YosefLab/SymSim"); library(SymSim)  # then run TrueCounts with tree-based USGeneEvfNet for each trajectory
```

## Evaluation signals

- Reproduced symsim_feature_selection_results.csv should show similarity-based methods (DELVE, Laplacian Score, Hotspot) with higher cell-type classification accuracy and pseudotime Kendall correlation than variance-based approaches
- DELVE/LS/Hotspot should remain robust at small feature subsets (e.g., 100 genes), while variance-based methods need ~2000 genes for comparable performance (consistent with Supplementary Fig. 13)
- Each simulated dataset must contain 10,000 cells, 20,000 genes, and 4 cell types with matching ground-truth labels and latent progression values
- Metrics must be aggregated as medians across the five tree trajectories for each method x alpha_mean x p combination

## Limitations

- Simulation parameters (sigma, alpha_mean, alpha_sd, depth) impose specific noise and capture-efficiency regimes; results may not transfer to other simulation regimes or technologies.
- Feature selection can greatly bias interpretation of the underlying cellular trajectory, so careful consideration is needed when interpreting benchmark outcomes.
- Results are specific to the UMI protocol setting; dropout-heavy non-UMI protocols may behave differently.

## Evidence

- [supplementary] Simulate five tree differentiation trajectories with SymSim v0.0.0.9000 in R v4.1.1 (10,000 cells, 20,000 genes, 4 cell types): [evidence span withheld — anchor status: not_found]
- [supplementary] Evaluate with 3-fold kNN and Kendall correlation workflow: [evidence span withheld — anchor status: not_found]
- [readme] SymSim installation instructions in the benchmark README: "library("devtools")
devtools::install_github("YosefLab/SymSim")"
- [readme] Benchmark purpose statement from the README: [evidence span withheld — anchor status: external_doc]
