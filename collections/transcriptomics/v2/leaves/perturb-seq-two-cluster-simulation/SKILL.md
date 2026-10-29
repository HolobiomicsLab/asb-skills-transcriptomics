---
name: perturb-seq-two-cluster-simulation
description: Use when you need a controlled two-cluster Perturb-seq benchmark with
  known true cellular response to evaluate whether a decoupling method beats naive
  fold-change estimation under infected proportion bias.
when_to_use_negative:
- Your goal is real-data perturbation analysis rather than method benchmarking — the
  mixed K562/RPE1 simulation has synthetic ground truth that does not exist in real
  experiments.
- The system has more than two cell clusters or rare cell types completely absent
  from infection, where the article notes scDecouple's GMM and cluster-proportion
  assumptions may not hold.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: scDecouple
  role: R package providing data_preprocessing, pc_selection, and the GMM-EM scDecouple
    function used to estimate cellular response on the simulated two-cluster data
  repo: https://github.com/MengQiuchen/scDecouple
- name: R
  role: Execution environment for the simulation and MAE comparison workflow
- name: devtools
  role: Used to install the scDecouple package from GitHub (devtools::install_github)
- name: mixtools
  role: Supports Gaussian mixture model fitting in the EM decoupling step
- name: diptest
  role: Provides dip statistics used by PC selection to flag multimodal PCs with high
    infection bias
- name: Seurat
  role: Loaded for single-cell preprocessing support in the scDecouple demo workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:324a29c2c5b97a5f4e8fea76d3b5887978ae5f0417a2638867144bf3a7bab2d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MengQiuchen/scDecouple
  tools:
  - R
  - Seurat
  - devtools
  - diptest
  - enrichR
  - mixtools
  - scDecouple
  - tidyverse
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# perturb-seq-two-cluster-simulation

## Summary

Constructs a simulated two-cluster scCRISPR-seq perturbation group by mixing K562 and RPE1 cells from a genome-wide Perturb-seq study, providing known ground-truth cellular responses for benchmarking scDecouple against the naive observed-change (OC) estimator via MAE on PC 1.

## When to use

Use when you need a controlled two-cluster Perturb-seq benchmark with known true cellular response to evaluate whether a decoupling method beats naive fold-change estimation under infected proportion bias.

## When NOT to use

- Your goal is real-data perturbation analysis rather than method benchmarking — the mixed K562/RPE1 simulation has synthetic ground truth that does not exist in real experiments.
- The system has more than two cell clusters or rare cell types completely absent from infection, where the article notes scDecouple's GMM and cluster-proportion assumptions may not hold.

## Inputs

- Genome-wide Perturb-seq data for K562 and RPE1 cells (reference [15])
- Simulated perturbation group Y (cell-by-gene matrix)
- Simulated control group Z (cell-by-gene matrix)
- List of shared editing loci (168 loci) for error evaluation

## Outputs

- MAE of scDecouple-estimated cellular responses on PC 1 versus true response
- MAE of OC-estimated cellular responses on PC 1 versus true response
- Dropout robustness comparison results written to mae_comparison.csv

## How to apply

Load genome-wide Perturb-seq data for K562 and RPE1 cells and mix the two populations to construct a perturbation group Y and control group Z as cell-by-gene matrices. Preprocess with scDecouple's data_preprocessing to obtain log-normalized expression and highly variable genes, then run pc_selection on the control-group PCA with using_rank = TRUE. Apply the GMM-EM decoupling on PC 1 with K = 2 components (seed_ = 0) and OC on the remaining PCs, then compute mean absolute error of estimated cellular responses across the 168 shared editing loci. To stress robustness, introduce dropout (batch-effect) noise into the simulated data and repeat the scDecouple-vs-OC comparison; the simulation is designed so the infected cell cluster ratio changes substantially, which is where scDecouple's advantage over OC is greatest.

## Related tools

- **scDecouple** (R package providing data_preprocessing, pc_selection, and the GMM-EM scDecouple function used to estimate cellular response on the simulated two-cluster data) — https://github.com/MengQiuchen/scDecouple
- **R** (Execution environment for the simulation and MAE comparison workflow)
- **devtools** (Used to install the scDecouple package from GitHub (devtools::install_github))
- **mixtools** (Supports Gaussian mixture model fitting in the EM decoupling step)
- **diptest** (Provides dip statistics used by PC selection to flag multimodal PCs with high infection bias)
- **Seurat** (Loaded for single-cell preprocessing support in the scDecouple demo workflow)

## Examples

```
res <- run_scDecouple(Y, Z)
```

## Evaluation signals

- scDecouple MAE on PC 1 cellular responses across the 168 shared editing loci is lower than OC MAE on the same simulated data
- The scDecouple advantage over OC is greatest in simulation settings where the infected cells show a substantial cluster ratio change
- Preprocessing yields log-normalized expression and highly variable genes; pc_selection (using_rank = TRUE) returns PC 1 as a bias-affected, multimodal PC (dip statistic above threshold)
- GMM-EM decoupling runs with K = 2 components and seed_ = 0 without errors; results are reproducible across runs with the same seed
- mae_comparison.csv contains both the primary MAE comparison and the dropout (batch-effect noise) robustness check

## Limitations

- The GMM assumption on expression distributions may not hold in intricate systems or disease tissues such as tumors
- The assumption of the same cell clusters between NT and perturbation groups may fail when rare cell types are completely absent from infection
- Assumption of similar cellular responses across cell clusters limits scDecouple to cells with moderate heterogeneity

## Evidence

- [results] Two-cluster K562/RPE1 simulation setup: "We validated the efficacy of scDecouple through a series of simulations"
- [abstract] scDecouple modeling approach: "scDecouple first models the distribution of gene expression profiles in perturbed cells and then iteratively finds the maximum likelihood of cell cluster proportions as well as the cellular response"
- [readme] Step-by-step decoupling call: "res.step3 <- scDecouple(Z.pca.mat = res.step1$Z.pca.results$x, Z.pca.rotation = res.step1$Z.pca.results$rotation, Y.norm.log.sub = res.step1$Y.norm.log.sub, select.pcs = res.step2)"
- [methods] PC selection with rank-based thresholds: "res.step2 <- pc_selection(Z.pca.mat = res.step1$Z.pca.results$x, Z.pca.sdev = res.step1$Z.pca.results$sdev, using_rank = TRUE)"
- [readme] Installation of package: "devtools::install_github("MengQiuchen/scDecouple", build_vignettes = TRUE)"
