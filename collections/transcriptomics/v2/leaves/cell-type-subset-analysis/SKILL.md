---
name: cell-type-subset-analysis
description: Use when benchmarking phase inference on heterogeneous droplet-based
  scRNA-seq where per-cell-type accuracy (e.g., median phase error per population)
  must be computed against known sampling times.
when_to_use_negative:
- The dataset lacks cell-type annotations, so no meaningful SMC/fibroblast/endothelial/macrophage
  subsets can be formed and only whole-population analysis (as done for the hepatocyte
  time course) is possible
- The input is plate-based scRNA-seq with a different generative model and the goal
  is not per-cell-type benchmarking on droplet-based data
- The analysis goal is a single pooled phase estimate across the entire tissue rather
  than cell-type-resolved phase errors and Bayes factors
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Tempo
  role: Bayesian phase-inference method run on each cell-type subset to produce per-cell
    circadian phase estimates
  repo: https://github.com/bauerbach95/tempo
- name: scanpy
  role: anndata-based loading and subsetting of the aorta scRNA-seq data by cell type
- name: Cyclops
  role: competing phase-inference method run on the same cell-type subsets for benchmarking
- name: Cyclum
  role: competing phase-inference method run on the same cell-type subsets for benchmarking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:f59f72e678c13ecc45d5f316d06efea6be7b1e6ffdfe2a48549895c44bc2a907
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bauerbach95/tempo
  tools:
  - Cyclops
  - Cyclum
  - Tempo
  - anndata
  - numpy
  - pandas
  - scanpy
  - scikit-learn
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cell-type-subset-analysis

## Summary

Stratifies droplet-based scRNA-seq data by cell type (e.g., subsetting the 10X Chromium mouse aorta light-dark dataset to SMCs, fibroblasts, endothelial cells, and macrophages as anndata objects) so that circadian phase inference methods like Tempo can be benchmarked per population against known sampling-time references.

## When to use

Use when benchmarking phase inference on heterogeneous droplet-based scRNA-seq where per-cell-type accuracy (e.g., median phase error per population) must be computed against known sampling times.

## When NOT to use

- The dataset lacks cell-type annotations, so no meaningful SMC/fibroblast/endothelial/macrophage subsets can be formed and only whole-population analysis (as done for the hepatocyte time course) is possible
- The input is plate-based scRNA-seq with a different generative model and the goal is not per-cell-type benchmarking on droplet-based data
- The analysis goal is a single pooled phase estimate across the entire tissue rather than cell-type-resolved phase errors and Bayes factors

## Inputs

- 10X Chromium mouse aorta light-dark scRNA-seq dataset as an anndata (.h5ad) object with cell-type annotations in adata.obs
- Droin et al. hepatocyte time-course scRNA-seq dataset as an anndata object
- Cell-type label column for SMCs, fibroblasts, endothelial cells, and macrophages
- Known sampling-time reference labels for computing per-cell phase errors

## Outputs

- Per-cell-type anndata subsets (SMCs, fibroblasts, endothelial cells, macrophages) ready for phase inference
- Per-cell phase estimates in hours for each method x cell-type combination
- phase_benchmark_results.csv containing median phase errors and log Bayes factors per method per cell type

## How to apply

Load the raw 10X Chromium mouse aorta light-dark scRNA-seq dataset and the Droin et al. hepatocyte time-course dataset as anndata objects in the tempo conda environment (Python >= 3.8, scanpy>=1.6, pytorch>=1.9.0, anndata, numpy, pandas, scikit-learn, scipy). Use the cell-type annotations in adata.obs to subset the aorta data to smooth muscle cells (SMCs), fibroblasts, endothelial cells, and macrophages, keeping each subset as a separate anndata object; the hepatocyte time course is analyzed as a single population. Run each phase-inference method (Tempo, Cyclops, Cyclum, and PCA with the core clock gene set and with all genes) independently on each cell-type subset and record per-cell phase estimates in hours. Compute the median phase error of each method-cell-type combination against the known sampling-time reference and the Bayes factor of Tempo versus each competitor per cell type, then assemble results into a single table (phase_benchmark_results.csv). This per-subset design is essential because the paper evaluates phase accuracy separately per cell type (e.g., Tempo all-genes SMCs = 1.88 h), and cell-type-specific CCG identities vary across populations.

## Related tools

- **Tempo** (Bayesian phase-inference method run on each cell-type subset to produce per-cell circadian phase estimates) — https://github.com/bauerbach95/tempo
- **scanpy** (anndata-based loading and subsetting of the aorta scRNA-seq data by cell type)
- **Cyclops** (competing phase-inference method run on the same cell-type subsets for benchmarking)
- **Cyclum** (competing phase-inference method run on the same cell-type subsets for benchmarking)

## Examples

```
conda activate tempo; adata = sc.read_h5ad('aorta_ld.h5ad'); sub = adata[adata.obs['cell_type'].isin(['SMC','fibroblast','endothelial','macrophage'])]
```

## Evaluation signals

- Reproduced median phase errors match Table 1, e.g., Tempo all-genes SMCs = 1.88 h on the aorta dataset
- Bayes factors reproduce the reported orders of magnitude: 10^17896 (SMCs), 10^2257 (fibroblasts), 10^110 (endothelial cells), 10^184 (macrophages), 10^1761 (hepatocytes)
- run_test.py in the tempo environment prints 'SUCCESSFULLY FINISHED' before the benchmark is launched
- Each cell-type subset retains its per-cell sampling-time labels so median phase errors in hours can be computed against the reference
- phase_benchmark_results.csv contains one median-phase-error and log-Bayes-factor row per method x cell-type combination (Tempo, Cyclops, Cyclum, PCA; core clock gene set and all genes)

## Limitations

- Existing methods (Cyclops, Cyclum, PCA) yield poor point estimates on droplet-based scRNA-seq because their transcript likelihood distributions poorly approximate the true generative distribution, so per-cell-type comparisons are strongly favorable to Tempo
- Tempo assumes 24 h sinusoidal CCG expression means; subpopulation-specific gene amplitude variation is not explicitly modeled, which can bias phase estimates within a subset
- Explicit modeling of technical effects becomes necessary when data are collected as multiple samples over time, affecting time-course subsets

## Evidence

- [other] subsetting the aorta data to SMCs, fibroblasts, endothelial cells, and macrophages: [evidence span withheld — anchor status: not_found]
- [readme] Tempo requires python >= 3.8 and depends on the following python packages: "Tempo requires python >= 3.8 and depends on the following python packages"
- [intro] Relative to droplet-based techniques (e.g., 10X Genomics Chromium), plate-based approaches tend to capture fewer cells: "Relative to droplet-based techniques (e.g., 10X Genomics Chromium), plate-based approaches tend to capture fewer cells"
- [other] median phase errors as low as 1.88 h for aorta SMCs (Table 1): [evidence span withheld — anchor status: not_found]
