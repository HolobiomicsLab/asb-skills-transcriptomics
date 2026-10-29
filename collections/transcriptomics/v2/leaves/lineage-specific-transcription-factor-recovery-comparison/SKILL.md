---
name: lineage-specific-transcription-factor-recovery-comparison
description: Use when comparing feature selection strategies on single-cell RNA-seq
  trajectories by counting lineage-specific transcription factors recovered per method,
  e.g., to show DELVE outperforms HVG and RNA velocity selection.
when_to_use_negative:
- Input is a protein iterative immunofluorescence dataset with no gene-level TF annotation
  available from AnimalTFDB
- The biological system lacks a known lineage-specific transcription factor ground
  truth, making the TF-recovery comparison uninformative
- Only a single feature selection method is of interest with no benchmark against
  HVG or RNA velocity alternatives
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: DELVE
  role: Unsupervised feature selection identifying dynamic modules and the dynamic
    seed gene set ranked by Laplacian score
  repo: https://github.com/jranek/delve
- name: delve_benchmark
  role: Provides fs class and run_delve_fs / laplacian_score_fs / hvg functions to
    perform and compare feature selection methods
  repo: https://github.com/jranek/delve_benchmark
- name: SketchKH
  role: Kernel Herding sketching of representative cell neighborhoods (num_subsamples=1000)
    prior to DELVE clustering
  repo: https://github.com/CompCy-lab/SketchKH
- name: Scanpy
  role: Diffusion pseudotime trajectory inference (dpt, 20 diffusion components, k=10)
    rooted in day-0 cells
- name: ScVelo
  role: RNA velocity dynamical model providing gene likelihood rankings as the competing
    feature set
- name: statsmodels
  role: Negative binomial generalized additive model regressing each of the 500 genes
    along estimated pseudotime
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:473076c57dc9c278a32d0b333a6691674ae39749f72573d38917b3578d6b86c1
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.631578947368421
n_claims: 19
---

# lineage-specific-transcription-factor-recovery-comparison

## Summary

Benchmarks feature selection methods (DELVE, HVG, RNA velocity) on their ability to recover lineage-specific transcription factors driving cell fate specification along the hESC-to-definitive-endoderm trajectory, quantifying method differences via pseudotime regression and AnimalTFDB annotation.

## When to use

Use when comparing feature selection strategies on single-cell RNA-seq trajectories by counting lineage-specific transcription factors recovered per method, e.g., to show DELVE outperforms HVG and RNA velocity selection.

## When NOT to use

- Input is a protein iterative immunofluorescence dataset with no gene-level TF annotation available from AnimalTFDB
- The biological system lacks a known lineage-specific transcription factor ground truth, making the TF-recovery comparison uninformative
- Only a single feature selection method is of interest with no benchmark against HVG or RNA velocity alternatives

## Inputs

- Preprocessed hESC-to-definitive-endoderm differentiation h5ad object (5397 cells x 2000 genes, CPM normalized, log+1, HVG-filtered) from Zenodo record 10534873
- AnimalTFDB transcription factor annotation database
- Day-0 cell labels for pseudotime rooting

## Outputs

- Ranked top-500 gene lists per method (DELVE dynamic-seed Laplacian ranking, HVG, RNA velocity likelihood)
- Table of significant trajectory-associated genes per method (BH q < 0.05 negative binomial GAM)
- Comparison table of lineage-specific transcription factor counts recovered by DELVE, HVG, and RNA velocity
- Pseudotime traces of key regulators (SOX2, POU5F1, EOMES, GSC, SOX17, FOXA2) per method as in Fig. 6h

## How to apply

Load the preprocessed definitive-endoderm differentiation .h5ad object (5397 cells x 2000 genes, CPM normalized, log+1, HVG-filtered) from Zenodo record 10534873 and run DELVE (delve_fs) on PCA-reduced data (n_pcs=50) with a k=10 nearest-neighbor affinity graph, Kernel Herding sketching (num_subsamples=1000), and KMeans++ clustering into c=5 feature modules repeated across 10 random initializations, taking the dynamic seed as the intersection across runs. Rank all features by ascending Laplacian score on the affinity graph rebuilt from the dynamic seed genes and retain the top p=500; in parallel select the top 500 HVGs and rank genes by RNA velocity likelihood from the ScVelo dynamical model. For each feature set, infer the trajectory with diffusion pseudotime (Scanpy dpt, 20 diffusion components, k=10) rooted in day-0 cells, then regress each of the 500 genes along pseudotime with a negative binomial GAM (statsmodels), calling genes significant at Benjamini-Hochberg q < 0.05. Annotate significant lineage-specific genes as transcription factors using AnimalTFDB and tabulate the counts recovered by DELVE, HVG, and RNA velocity; the expected result is that DELVE recovers ~30% more lineage-specific transcription factors, as reported in Fig. 6g.

## Related tools

- **DELVE** (Unsupervised feature selection identifying dynamic modules and the dynamic seed gene set ranked by Laplacian score) — https://github.com/jranek/delve
- **delve_benchmark** (Provides fs class and run_delve_fs / laplacian_score_fs / hvg functions to perform and compare feature selection methods) — https://github.com/jranek/delve_benchmark
- **SketchKH** (Kernel Herding sketching of representative cell neighborhoods (num_subsamples=1000) prior to DELVE clustering) — https://github.com/CompCy-lab/SketchKH
- **Scanpy** (Diffusion pseudotime trajectory inference (dpt, 20 diffusion components, k=10) rooted in day-0 cells)
- **ScVelo** (RNA velocity dynamical model providing gene likelihood rankings as the competing feature set)
- **statsmodels** (Negative binomial generalized additive model regressing each of the 500 genes along estimated pseudotime)

## Examples

```
kh_indices, adata_subsample = sketch(adata, sample_set_key='day', gamma=1, num_subsamples=1000, frequency_seed=0, n_jobs=-1); predicted_features = run_delve_fs(adata_subsample, num_pc=50, num_neighbors=10, num_clusters=5, num_runs=10)
```

## Evaluation signals

- DELVE lineage-specific transcription factor count must exceed HVG and RNA velocity counts by approximately 30%, matching Fig. 6g
- DELVE recovers two dynamic modules corresponding to pluripotency/primitive streak loss and DE formation
- All genes called significant must pass Benjamini-Hochberg q < 0.05 in the negative binomial GAM pseudotime regression
- Pseudotime traces of key regulators (SOX2, POU5F1 down; EOMES, GSC, SOX17, FOXA2 up) reproduce Fig. 6h patterns
- Each feature set contains exactly 500 genes (p=500) ranked per the specified selection criteria

## Limitations

- Feature selection can greatly bias interpretation of the underlying cellular trajectory; careful consideration should be made when performing feature selection for trajectory analysis
- DELVE could be improved by incorporating a set of previously established regulators
- All dynamic modules were leveraged equally for constructing the approximate cellular trajectory; excluding unwanted dynamic modules is possible but not implemented

## Evidence

- [results] DELVE feature selection recovers more TFs: [evidence span withheld — anchor status: not_found]
- [discussion] Workflow definition: "performing feature selection with DELVE prior to performing trajectory inference improved the identification and resolution of gene programs"
- [abstract] DELVE method description: "DELVE uses a bottom-up approach to mitigate the effects of confounding sources of variation, and instead models cell states from dynamic gene or protein modules based on core regulatory complexes"
- [readme] Data access: "You can download all of the preprocessed single-cell datasets (`.h5ad` files) from the [Zenodo](https://zenodo.org/records/10534873) repository."
- [discussion] Laplacian ranking context: "similarity-based feature selection methods (e.g., DELVE, Laplacian score, and Hotspot) achieved the highest recovery of differentially expressed genes within a cell type or along a cellular lineage"
