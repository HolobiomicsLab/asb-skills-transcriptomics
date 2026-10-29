---
name: single-cell-clustering-benchmarking
description: Use when comparing a rare-cell or sub-cluster reclustering method against
  baseline scRNA-seq clusterers on datasets with known or suspected rare populations,
  e.g. PBMC3k or simulated pancreas data with rare closely placed clusters.
when_to_use_negative:
- The goal is annotating an already-accepted clustering with marker genes rather than
  comparing competing clustering methods
- The input is not scRNA-seq data (e.g. copy-number or B-allele-frequency tables for
  CONICS-style CNV calling)
- Only one clustering exists and there is no second method or ground truth to compare
  against
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SCISSORS
  role: Primary reclustering method under benchmark; provides ComputeSilhouetteScores
    and CosineDist for silhouette-based evaluation
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Baseline Louvain clustering (resolution sweep 0.5–2.0) and preprocessing infrastructure
  repo: https://github.com/jr-leary7/SCISSORS
- name: SAFEclustering
  role: Ensemble competitor via individual_clustering (SC3 + Seurat + tSNE, nPC.seurat=15)
    followed by SAFE (HGPA, MCLA, CSPA), SEED=629
- name: giniclust3
  role: Rare-cell competitor run in Python (calGini/clusterGini with 7 neighbors,
    calFano/clusterFano, generateMtilde/clusterMtilde consensus)
- name: cluster
  role: Provides cluster::silhouette for reference silhouette computation on cosine-distance
    matrices
- name: ggplot2
  role: Violin plots of per-method silhouette distributions (draw_quantiles = 0.5)
    and resolution-sweep line plots
- name: ggsignif
  role: Maps paired wilcox.test significance levels onto comparison bars between SCISSORS
    and each competitor
- name: MultiK
  role: Alternative consensus cluster-number estimator benchmarked in the comparison
    (mk <- MultiK(seu = pbmc_raw, seed = 312))
provenance:
  source_task_ids:
  - task_003
  - task_002
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:a013c2c55197fd21f13d19bb7f87aca84808efdaf51f93e1024eb3f3f8a64213
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jr-leary7/SCISSORS
  tools:
  - MultiK
  - R
  - SAFEclustering
  - SCISSORS
  - Seurat
  - SeuratData
  - cluster
  - dplyr
  - ggplot2
  - ggsignif
  - giniclust3
  - magrittr
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# single-cell-clustering-benchmarking

## Summary

Benchmark SCISSORS reclustering against alternative single-cell clustering methods (Seurat Louvain, SAFE-clustering ensembles, GiniClust3, MultiK) using per-cell silhouette scores in PCA space and external agreement metrics (ARI/NMI) against ground-truth labels.

## When to use

Use when comparing a rare-cell or sub-cluster reclustering method against baseline scRNA-seq clusterers on datasets with known or suspected rare populations, e.g. PBMC3k or simulated pancreas data with rare closely placed clusters.

## When NOT to use

- The goal is annotating an already-accepted clustering with marker genes rather than comparing competing clustering methods
- The input is not scRNA-seq data (e.g. copy-number or B-allele-frequency tables for CONICS-style CNV calling)
- Only one clustering exists and there is no second method or ground truth to compare against

## Inputs

- Preprocessed scRNA-seq Seurat object (e.g. PBMC3k from SeuratData, filtered for nFeature_RNA < 2500)
- Simulated pancreas scRNA-seq count matrix (996 cells, 15,270 genes) with 7 ground-truth cluster labels
- PCA cell embeddings per method for distance computation
- Per-method cluster label vectors

## Outputs

- Per-cell and per-cluster silhouette score dataframes for each method (e.g. silhouette_scores.csv)
- ARI/NMI metrics table comparing each method against ground-truth labels
- Resolution-sweep figure of ARI, NMI, and silhouette vs Louvain resolution (0.5–2.0) saved as PDF
- ggplot2 violin plot of per-method silhouette distributions with ggsignif wilcox.test significance bars

## How to apply

Preprocess each dataset per method-appropriate pipelines (e.g. SCISSORS::PrepareData with n.HVG=4000, n.PC=15, initial.resolution=0.4, random.seed=629 on PBMC3k; Seurat NormalizeData/vst-2000-variable-features/RunPCA/FindNeighbors dims 1:10/FindClusters resolution=0.5 for the baseline copy). Run each candidate method on the same preprocessed input, including a Seurat Louvain resolution sweep from 0.5 to 2.0. Compute per-cell silhouette scores with SCISSORS::ComputeSilhouetteScores(avg=FALSE) or cluster::silhouette on a CosineDist cosine-distance matrix of the PCA embeddings, and for labeled datasets compute ARI and NMI against the ground-truth clusters. Summarize each method with median/aggregate silhouette and plot violin distributions (draw_quantiles=0.5) with ggsignif paired wilcox.test significance bars; for simulated data (996 cells, 15,270 genes, 7 true clusters) additionally plot ARI, NMI, and silhouette versus Louvain resolution to show whether increasing resolution improves clustering.

## Related tools

- **SCISSORS** (Primary reclustering method under benchmark; provides ComputeSilhouetteScores and CosineDist for silhouette-based evaluation) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Baseline Louvain clustering (resolution sweep 0.5–2.0) and preprocessing infrastructure) — https://github.com/jr-leary7/SCISSORS
- **SAFEclustering** (Ensemble competitor via individual_clustering (SC3 + Seurat + tSNE, nPC.seurat=15) followed by SAFE (HGPA, MCLA, CSPA), SEED=629)
- **giniclust3** (Rare-cell competitor run in Python (calGini/clusterGini with 7 neighbors, calFano/clusterFano, generateMtilde/clusterMtilde consensus))
- **cluster** (Provides cluster::silhouette for reference silhouette computation on cosine-distance matrices)
- **ggplot2** (Violin plots of per-method silhouette distributions (draw_quantiles = 0.5) and resolution-sweep line plots)
- **ggsignif** (Maps paired wilcox.test significance levels onto comparison bars between SCISSORS and each competitor)
- **MultiK** (Alternative consensus cluster-number estimator benchmarked in the comparison (mk <- MultiK(seu = pbmc_raw, seed = 312)))

## Examples

```
sil_score_df <- ComputeSilhouetteScores(seu_pbmc, avg = FALSE)
```

## Evaluation signals

- On the simulated pancreas dataset SCISSORS must attain ARI = 0.987, NMI = 0.985, and silhouette = 0.693, and be the only method that separates the two rare clusters
- The silhouette violin figure must show SCISSORS with the highest median silhouette and statistically significant paired wilcox.test differences versus Seurat, SAFE, and GiniClust3
- In the Louvain resolution sweep the lowest resolution (0.5) must show the highest ARI/NMI/silhouette, i.e. increasing resolution does not improve clustering
- All per-cell silhouette scores must lie within the valid range -1 to 1 and match the cluster::silhouette reference values within numerical tolerance
- All methods must be scored on the same PCA-space distances and the same cells (per-cell silhouette, avg = FALSE) for the comparison to be fair

## Limitations

- Small differences in analysis and parameter choice can greatly alter clustering results, so benchmark outcomes are sensitive to preprocessing settings such as n.HVG and n.PC
- Existing methods largely focus on determining the optimal number of robust clusters, which is problematic for extremely low-abundance cells due to their subtle contributions
- Baseline clusterings (SAFE, GiniClust3) require per-cell silhouette via cluster::silhouette on cosine distances rather than the SCISSORS-native ComputeSilhouetteScores, so cross-method comparability depends on using identical distance matrices

## Evidence

- [abstract] Silhouette scoring as heterogeneity estimate: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process"
- [abstract] Sensitivity of results to parameters: "even small differences in analysis and parameter choosing can greatly alter clustering results and thus impose great influence on which cell types are identified"
- [methods] Silhouette computation call: "sils_SCISSORS <- ComputeSilhouetteScores(pbmc_SCISSORS, avg = FALSE)"
- [readme] Benchmark PBMC3k setup: [evidence span withheld — anchor status: not_found]
