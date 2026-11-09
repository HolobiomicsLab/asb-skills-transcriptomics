---
name: dynamic-feature-module-permutation-testing
description: Use when snapshot scRNA-seq data must be reduced to dynamically covarying
  gene modules before trajectory inference, and noisy or low-variance lineage-specific
  features (e.g., transcription factors) would otherwise be missed by HVG or variance-based
  selection.
when_to_use_negative:
- 'The task is supervised feature selection: cell-type annotations are already available
  and supervised methods can exploit them directly, whereas DELVE is designed for
  the unsupervised setting where manual annotations are unavailable'
- The input is already a curated feature set or previously established regulator list
  — the article notes DELVE could instead be improved by incorporating such established
  regulators rather than rediscovering them
- The analysis goal requires retaining all sources of variation, since feature selection
  can greatly bias interpretation of the underlying cellular trajectory
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DELVE
  role: Unsupervised feature selection (delve_fs) that clusters dynamic feature modules
    and ranks features via Laplacian score on the seed graph
  repo: https://github.com/jranek/delve
- name: SketchKH
  role: Kernel Herding sketching (num_subsamples=1000) of representative cell neighborhoods
    prior to module discovery
  repo: https://github.com/CompCy-lab/SketchKH
- name: delve_benchmark
  role: Provides run_delve_fs and evaluation functions to compare module-derived feature
    selection against HVG, Laplacian score, and other methods
  repo: https://github.com/jranek/delve_benchmark
- name: anndata
  role: Reading and manipulating the preprocessed .h5ad single-cell data object
- name: scikit-learn
  role: PCA reduction and KMeans++ clustering used in the module discovery step
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:b4be7f24c2150f68277e8c5f93ddf13e7d529600d5f1507bc0d6900b6015cbb7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve
  tools:
  - DELVE
  - EnrichR
  - Python
  - ScVelo
  - Scanpy
  - SketchKH
  - anndata
  - delve_benchmark
  - gseapy
  - scikit-learn
  - statsmodels
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# dynamic-feature-module-permutation-testing

## Summary

Identifies dynamic gene modules and a robust dynamic seed feature set from single-cell data by running DELVE's unsupervised module discovery (kNN affinity graph, Kernel Herding sketching, KMeans++ clustering) repeated across random initializations and retaining only features consistently assigned across runs, to support trajectory-preserving feature selection.

## When to use

Use when snapshot scRNA-seq data must be reduced to dynamically covarying gene modules before trajectory inference, and noisy or low-variance lineage-specific features (e.g., transcription factors) would otherwise be missed by HVG or variance-based selection.

## When NOT to use

- The task is supervised feature selection: cell-type annotations are already available and supervised methods can exploit them directly, whereas DELVE is designed for the unsupervised setting where manual annotations are unavailable
- The input is already a curated feature set or previously established regulator list — the article notes DELVE could instead be improved by incorporating such established regulators rather than rediscovering them
- The analysis goal requires retaining all sources of variation, since feature selection can greatly bias interpretation of the underlying cellular trajectory

## Inputs

- Preprocessed single-cell RNA-seq annotated data object (.h5ad, cells x genes; e.g., 5397 cells x 2000 genes, CPM normalized, log+1, HVG-filtered)
- PCA-reduced expression matrix (n_pcs=50)
- Optional sample-set key in adata.obs for Kernel Herding sketching

## Outputs

- Dynamic feature module assignments (e.g., c=5 modules per clustering run)
- Robust dynamic seed gene set defined as the intersection across 10 random clustering runs
- Ranked feature list by ascending Laplacian score on the seed-derived cell affinity graph (top p=500 genes)
- Comparison of recovered lineage-specific transcription factors against HVG and RNA velocity selections

## How to apply

Load a preprocessed .h5ad object (e.g., CPM-normalized, log1p-transformed, HVG-filtered) and reduce with PCA (n_pcs=50). Build a k=10 nearest-neighbor affinity graph, sketch representative cell neighborhoods with Kernel Herding (num_subsamples=1000, via sketchKH), and cluster features with KMeans++ into c=5 candidate modules. Repeat the clustering across 10 random initializations and define the dynamic seed as the intersection of features across runs, which acts as the robustness (permutation-style) test separating reproducible dynamic modules from initialization artifacts. Rebuild the between-cell affinity graph from the seed genes, rank all features by ascending Laplacian score, and retain the top p=500 genes for downstream trajectory inference. Rationale: DELVE's bottom-up modeling of dynamic feature modules constituting core regulatory complexes mitigates confounding variation that overwhelms variance-based or full-graph similarity-based feature selection.

## Related tools

- **DELVE** (Unsupervised feature selection (delve_fs) that clusters dynamic feature modules and ranks features via Laplacian score on the seed graph) — https://github.com/jranek/delve
- **SketchKH** (Kernel Herding sketching (num_subsamples=1000) of representative cell neighborhoods prior to module discovery) — https://github.com/CompCy-lab/SketchKH
- **delve_benchmark** (Provides run_delve_fs and evaluation functions to compare module-derived feature selection against HVG, Laplacian score, and other methods) — https://github.com/jranek/delve_benchmark
- **anndata** (Reading and manipulating the preprocessed .h5ad single-cell data object)
- **scikit-learn** (PCA reduction and KMeans++ clustering used in the module discovery step)

## Examples

```
pip install delve-fs
```

## Evaluation signals

- The dynamic seed gene set (intersection across 10 random KMeans++ clustering runs) is non-empty and stable across repeated initializations
- Recovered modules reproduce known biology on the hESC-to-definitive-endoderm dataset: one module corresponding to pluripotency/primitive streak loss and one to DE formation
- Genes significant along diffusion pseudotime (Benjamini-Hochberg q < 0.05, negative binomial GAM) include expected regulators: SOX2, POU5F1 (pluripotency) and EOMES, GSC, SOX17, FOXA2 (DE formation)
- The number of AnimalTFDB-annotated lineage-specific transcription factors recovered from DELVE-selected features exceeds HVG and RNA velocity selections by approximately 30%
- Trajectory inferred from DELVE-selected features (Scanpy dpt, 20 diffusion components, k=10, day-0 root) is at least as coherent as trajectories from the full feature set

## Limitations

- Feature selection can greatly bias the interpretation of the underlying cellular trajectory, so careful consideration is needed before applying it prior to trajectory inference
- All dynamic modules were leveraged equally for constructing the approximate cellular trajectory; excluding unwanted dynamic modules is possible but not implemented
- DELVE could be improved by incorporating a set of previously established regulators, and future work includes deconvolving trajectories using biological system-specific seed graphs

## Evidence

- [abstract] DELVE uses a bottom-up approach to mitigate the effects of confounding sources of variation by modeling cell states from dynamic gene or protein modules based on core regulatory complexes: [evidence span withheld — anchor status: not_found]
- [readme] In contrast to previous work, DELVE uses a bottom-up approach to mitigate the effect of unwanted sources of feature variation confounding inference: "In contrast to previous work, DELVE uses a bottom-up approach to mitigate the effect of unwanted sources of feature variation confounding inference, and instead models cell states from dynamic"
- [discussion] performing feature selection with DELVE prior to performing trajectory inference improved the identification and resolution of gene programs: "performing feature selection with DELVE prior to performing trajectory inference improved the identification and resolution of gene programs"
- [discussion] similarity-based feature selection methods (e.g., DELVE, Laplacian score, and Hotspot) achieved the highest recovery of differentially expressed genes: "similarity-based feature selection methods (e.g., DELVE, Laplacian score, and Hotspot) achieved the highest recovery of differentially expressed genes within a cell type or along a cellular lineage"
- [intro] they can be overwhelmed by noisy or irrelevant features that dominate data variance, and (2) are insensitive to lineage-specific features (e.g., transcription factors): "they can be overwhelmed by noisy or irrelevant features that dominate data variance, and (2) are insensitive to lineage-specific features (e.g., transcription factors) that have a small variance and"
