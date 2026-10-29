---
name: symsim-parameter-configuration
description: Use when you need ground-truth single-cell RNA-seq benchmark data with
  known cell-type labels and latent pseudotime to evaluate whether feature selection
  methods preserve cell types and cellular trajectories.
when_to_use_negative:
- You need real experimental single-cell data rather than simulations with known ground
  truth
- Your platform is non-UMI (e.g., plate-based full-length protocols) — the configured
  depth_mean = 5e5 and protocol = UMI parameters reflect the paper's UMI setting
- You only need to apply DELVE for feature selection rather than generate simulated
  benchmark data (use the delve package directly)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SymSim
  role: Simulates tree differentiation trajectories with tunable parameters (sigma,
    alpha_mean, alpha_sd, evf_type, protocol, depth) to produce ground-truth scRNA-seq
    benchmarks
  repo: https://github.com/jranek/delve_benchmark
- name: DELVE
  role: One of the feature selection methods evaluated on the simulated trajectories;
    similarity-based methods (DELVE, Laplacian Score, Hotspot) are expected to be
    robust at small feature subsets
  repo: https://github.com/jranek/delve
- name: delve_benchmark
  role: Python package providing the fs class and evaluation functions used to assess
    feature selection performance on the SymSim-simulated data
  repo: https://github.com/jranek/delve_benchmark
- name: Python
  role: Hosts the downstream preprocessing (PCA, n_pcs = 50) and evaluation (kNN classification,
    Kendall rank correlation) of features selected from the simulated data
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:8214cf33a5e74250f8be60b834be5d1b6e91a9319be68e300ddd67b5f223f0cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve_benchmark
  tools:
  - DELVE
  - Python
  - SymSim
  - delve_benchmark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# symsim-parameter-configuration

## Summary

Configure and run SymSim v0.0.0.9000 in R to simulate large-scale single-cell RNA-seq tree differentiation trajectories (10,000 cells, 20,000 genes, 4 cell types) with known ground-truth cell types and latent progression, providing controlled benchmarks for evaluating trajectory-preserving feature selection methods.

## When to use

Use when you need ground-truth single-cell RNA-seq benchmark data with known cell-type labels and latent pseudotime to evaluate whether feature selection methods preserve cell types and cellular trajectories.

## When NOT to use

- You need real experimental single-cell data rather than simulations with known ground truth
- Your platform is non-UMI (e.g., plate-based full-length protocols) — the configured depth_mean = 5e5 and protocol = UMI parameters reflect the paper's UMI setting
- You only need to apply DELVE for feature selection rather than generate simulated benchmark data (use the delve package directly)

## Inputs

- SymSim parameter set (sigma, alpha_mean vector, alpha_sd, evf_type, protocol, depth_mean, depth_sd)
- R v4.1.1 environment with SymSim v0.0.0.9000 (installed via devtools from YosefLab/SymSim)
- Trajectory tree specification defining five differentiation tree trajectories with 4 cell types

## Outputs

- Five simulated scRNA-seq count matrices (10,000 cells × 20,000 genes each)
- Ground-truth cell-type labels and latent progression values per cell
- symsim_feature_selection_results.csv with median classification accuracy and pseudotime correlation per method × α_mean × p combination

## How to apply

Install SymSim v0.0.0.9000 in R v4.1.1 via devtools from the YosefLab GitHub repository. Simulate five tree differentiation trajectories with parameters σ = 0.6 and 0.4, α_mean = (0.01, 0.02, 0.03, 0.04, 0.05), α_sd = 0.02, evf_type = continuous, protocol = UMI, depth_mean = 5e5 and depth_sd = 3e4, yielding 10,000 cells and 20,000 genes across 4 cell types with reduced mRNA capture efficiency. The varying α_mean values control the dynamism of gene expression along each trajectory, and the UMI protocol with the specified depth parameters induces realistic dropout. Export the simulated count matrices together with cell-type labels and ground-truth latent progression so that downstream preprocessing (PCA, n_pcs = 50) and feature selection evaluation (kNN classification, k = 3; Kendall rank correlation against diffusion pseudotime) can be performed in Python, aggregating metrics per method × α_mean × number of selected features.

## Related tools

- **SymSim** (Simulates tree differentiation trajectories with tunable parameters (sigma, alpha_mean, alpha_sd, evf_type, protocol, depth) to produce ground-truth scRNA-seq benchmarks) — https://github.com/jranek/delve_benchmark
- **DELVE** (One of the feature selection methods evaluated on the simulated trajectories; similarity-based methods (DELVE, Laplacian Score, Hotspot) are expected to be robust at small feature subsets) — https://github.com/jranek/delve
- **delve_benchmark** (Python package providing the fs class and evaluation functions used to assess feature selection performance on the SymSim-simulated data) — https://github.com/jranek/delve_benchmark
- **Python** (Hosts the downstream preprocessing (PCA, n_pcs = 50) and evaluation (kNN classification, Kendall rank correlation) of features selected from the simulated data)

## Examples

```
Rscript -e 'library("devtools"); devtools::install_github("YosefLab/SymSim")'
```

## Evaluation signals

- The reproduced symsim_feature_selection_results.csv shows DELVE, Laplacian Score, and Hotspot achieving higher cell-type classification accuracy and pseudotime correlation at small feature subsets (e.g., 100 genes) than variance-based approaches
- Variance-based methods require ~2000 genes to reach performance comparable to similarity-based methods, consistent with Supplementary Fig. 13
- Each simulated dataset contains 10,000 cells, 20,000 genes, and 4 cell types across five tree trajectories
- Kendall rank correlation between diffusion pseudotime and ground-truth latent progression is computable for every method × α_mean × p combination
- Aggregated metrics are medians across the five simulated datasets for each configuration

## Limitations

- Simulated parameters (UMI protocol, depth_mean = 5e5, specific α_mean values) may not generalize to all experimental designs or scRNA-seq protocols
- Feature selection can greatly bias interpretation of the underlying cellular trajectory; careful consideration is needed when performing feature selection for trajectory analysis

## Evidence

- [readme] SymSim install command: "#symsim v0.0.0.9000
library("devtools")
devtools::install_github("YosefLab/SymSim")"
- [discussion] Benchmark motivation: "we benchmarked twelve feature selection methods29,49,53,55–58 on their ability to identify biologically relevant features for trajectory analysis from single-cell RNA sequencing data and proteomic"
- [discussion] Similarity-based methods perform best: "similarity-based feature selection methods (e.g., DELVE, Laplacian score, and Hotspot) achieved the highest recovery of differentially expressed genes"
- [discussion] Feature selection bias caveat: "feature selection can greatly bias the interpretation of the underlying cellular trajectory45, thus careful consideration should be made when performing feature selection for trajectory analysis"
