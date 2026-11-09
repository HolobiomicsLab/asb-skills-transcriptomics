---
name: adjusted-rand-index-clustering-evaluation
description: Use when benchmarking scRNA-seq clustering or batch-effect correction
  outputs against known cell-type labels (e.g., FACS-sorted labels), especially when
  assembling an ARI/silhouette comparison table across methods.
when_to_use_negative:
- The experimental design is completely confounded (e.g., batch 1 measures cell types
  1 and 2, batch 2 measures cell types 3 and 4), where no batch-correction or clustering
  comparison method is applicable
- Reference cell-type labels are unavailable or unreliable, so chance-corrected clustering
  agreement against ground truth cannot be computed
- The goal is selecting the number of cell types K rather than benchmarking cluster
  quality against labels — BUSseq uses BIC comparison (K from 3 to 6) for that decision
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: BUSseq
  role: Bayesian hierarchical model (BUSseq_MCMC) producing cell-type cluster assignments
    and corrected values to be scored by ARI/silhouette
  repo: https://github.com/songfd2018/BUSseq-Rpackage
- name: Seurat
  role: Competing batch-correction pipeline whose cluster assignments are scored by
    ARI against the reference labels
- name: Scanorama
  role: Competing mutual-nearest-neighbor integration method whose clusters are scored
    by ARI
- name: LIGER
  role: Competing integration method whose clusters are scored by ARI
- name: scVI
  role: Competing deep-generative integration method whose clusters are scored by
    ARI
- name: ZINB-WaVE
  role: Competing batch-correction method whose clusters are scored by ARI
- name: MNN
  role: Competing mutual-nearest-neighbor method whose clusters are scored by ARI
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:1e62eae3bf9b1de1b269068bd20a1a7d91c7893ca6b80a753c1885c33edf4a51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq-Rpackage
  tools:
  - BASS
  - BUSseq
  - BayesSpace
  - ComBat
  - LIGER
  - MNN
  - R
  - RUV
  - SC.MEB
  - SVA
  - Scanorama
  - Seurat
  - SpaGCN
  - ZINB-WaVE
  - iSC.MEB
  - mclust
  - scMerge
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# adjusted-rand-index-clustering-evaluation

## Summary

Compute the Adjusted Rand Index (ARI) between a batch-correction method's inferred cell-type clusters and known reference labels (FACS-sorted or curated cell types) to quantify clustering consistency, as used to benchmark BUSseq against LIGER, MNN, Scanorama, scVI, Seurat, and ZINB-WaVE on hematopoietic and pancreas scRNA-seq data.

## When to use

Use when benchmarking scRNA-seq clustering or batch-effect correction outputs against known cell-type labels (e.g., FACS-sorted labels), especially when assembling an ARI/silhouette comparison table across methods.

## When NOT to use

- The experimental design is completely confounded (e.g., batch 1 measures cell types 1 and 2, batch 2 measures cell types 3 and 4), where no batch-correction or clustering comparison method is applicable
- Reference cell-type labels are unavailable or unreliable, so chance-corrected clustering agreement against ground truth cannot be computed
- The goal is selecting the number of cell types K rather than benchmarking cluster quality against labels — BUSseq uses BIC comparison (K from 3 to 6) for that decision

## Inputs

- Per-cell cluster assignments from each batch-correction method (e.g., BUSseq cell-type clustering output)
- Reference cell-type labels: FACS-sorted labels for hematopoietic data (GSE72857, GSE81682) or curated labels for pancreas data (GSE81076, GSE85241, GSE86473, E-MATB-5061)
- Corrected/imputed count values per method (e.g., corrected_read_counts(sce_BUSseqfit)) for silhouette computation
- Preprocessed common matrix of highly variable genes (3,470 genes hematopoietic; 7095 cells x 2480 genes pancreas)

## Outputs

- Per-method ARI values against reference labels (e.g., BUSseq 0.582 vs LIGER 0.307, MNN 0.575, Scanorama 0.518, scVI 0.197, Seurat 0.266, ZINB-WaVE 0.348 on hematopoietic data; BUSseq 0.608 on pancreas)
- Median silhouette coefficients on each method's corrected values
- Combined comparison table saved as batch_correction_benchmark.csv

## How to apply

Fit each batch-correction method (BUSseq via BUSseq_MCMC with K selected by BIC over K=3 to 6; competitors such as LIGER, MNN, Scanorama, scVI, Seurat, ZINB-WaVE with their standard pipelines) on the same preprocessed read-count matrix of highly variable genes. Obtain each method's per-cell cluster assignments and compare them to the reference labels — FACS-sorted cell-type labels for the hematopoietic data (GSE72857, GSE81682) and curated labels for the four pancreas datasets (GSE81076, GSE85241, GSE86473, E-MATB-5061) — using the adjusted Rand index. Compute ARI per method on identical cells/genes, and complement it with median silhouette coefficients on each method's corrected values. Assemble the per-method ARI and silhouette values into a single comparison table (e.g., batch_correction_benchmark.csv). The rationale is that ARI is chance-corrected, so it fairly ranks methods whose cluster numbers and label permutations differ; BUSseq is validated by attaining the top ARI (0.582 hematopoietic, 0.608 pancreas) among the compared methods.

## Related tools

- **BUSseq** (Bayesian hierarchical model (BUSseq_MCMC) producing cell-type cluster assignments and corrected values to be scored by ARI/silhouette) — https://github.com/songfd2018/BUSseq-Rpackage
- **Seurat** (Competing batch-correction pipeline whose cluster assignments are scored by ARI against the reference labels)
- **Scanorama** (Competing mutual-nearest-neighbor integration method whose clusters are scored by ARI)
- **LIGER** (Competing integration method whose clusters are scored by ARI)
- **scVI** (Competing deep-generative integration method whose clusters are scored by ARI)
- **ZINB-WaVE** (Competing batch-correction method whose clusters are scored by ARI)
- **MNN** (Competing mutual-nearest-neighbor method whose clusters are scored by ARI)

## Examples

```
Rscript -e 'library(mclust); ari <- mclust::adjustedRandIndex(method_clusters, facs_labels); write.csv(data.frame(method, ari), "batch_correction_benchmark.csv", append=TRUE)'
```

## Evaluation signals

- Per-method ARI values reproduce the reported benchmark within tolerance (BUSseq 0.582 highest on hematopoietic data; 0.608 highest on pancreas data)
- BUSseq attains both the highest ARI and the highest median silhouette coefficient among all seven compared methods
- ARI is computed on identical preprocessed cells and genes (3,470 hematopoietic HVGs; 7095 cells x 2480 pancreas HVGs) for all methods, ensuring fair comparison
- ARI is invariant to label permutation between inferred clusters and reference labels (a sanity check on the metric implementation)
- Final comparison table batch_correction_benchmark.csv is assembled with ARI and silhouette per method and the top row is BUSseq

## Limitations

- MCMC inference is computationally heavy; the article notes algorithms are 'well-known for heavy computation load', mitigated only by parallelization across cores (e.g., ~12 mins for 4 K values in the demo; 500 iterations on four cores per K in the benchmark)
- Multi-stage pipelines that cluster first and then use clusters downstream ignore clustering uncertainty, which can lead to false positives and false negatives — ARI against noisy labels inherits this
- A completely confounded design admits no applicable method, so ARI comparison is undefined in that regime

## Evidence

- [abstract] We demonstrate that BUSseq outperforms existing methods with simulated and real data.: "We demonstrate that BUSseq outperforms existing methods with simulated and real data."
- [methods] Corrected values used as if single batch: "the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch"
- [methods] BIC-based K selection: "To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6."
- [readme] BUSseq capabilities from README: "BUSseq is able to simultaneously correct batch effects, clusters cell types, and takes care of the count data nature, the overdispersion, the dropout events, and the cell-specific sequencing depth of"
