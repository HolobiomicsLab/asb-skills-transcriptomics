---
name: ari-nmi-clustering-accuracy-evaluation
description: Use when comparing cell-type clustering results against retained ground-truth
  labels on simulated scRNA-seq data — varying cluster numbers, rare-cell proportions,
  and dropout — to quantify agreement via ARI, NMI, and silhouette scores.
when_to_use_negative:
- The dataset has no retained ground-truth cell-type labels — ARI and NMI require
  a known reference partition and cannot be computed from clusters alone
- The analysis goal is rare-cell subcluster discovery on real (unlabeled) data rather
  than benchmarking clustering accuracy on simulations — use SCISSORS's reclustering
  workflow directly instead
- You need to determine the optimal number of robust clusters rather than measure
  how well an existing partition recapitulates labels
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCISSORS
  role: Method under evaluation; run in completely unsupervised mode with silhouette-score-driven
    reclustering on each simulated dataset
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Standard preprocessing (NormalizeData, FindVariableFeatures with vst, ScaleData,
    RunPCA) and Louvain clustering baseline run on the same simulations
- name: giniclust3
  role: Competing baseline (Gini/Fano consensus clustering) run on identical simulated
    datasets for ARI/NMI comparison
- name: cluster
  role: Provides cluster::silhouette used to compute per-cell silhouette scores from
    CosineDist distance matrices
- name: dplyr
  role: Assembles the method-by-metric results dataframe of ARI, NMI, and silhouette
    values
- name: ggplot2
  role: Renders the Fig. 2a-c ARI/NMI bar panels and silhouette violin plots with
    ggsignif significance bars
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:1a067b0680569845046a228c0960e7584e3d2a862eb14a110376ea2cbed5dc27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jr-leary7/SCISSORS
  tools:
  - R
  - SCISSORS
  - Seurat
  - cluster
  - dplyr
  - ggplot2
  - ggsignif
  - giniclust3
  - magrittr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# ari-nmi-clustering-accuracy-evaluation

## Summary

Evaluates scRNA-seq clustering accuracy by computing Adjusted Rand Index (ARI), Normalized Mutual Information (NMI), and per-cell silhouette scores against retained ground-truth cell-type labels on simulated datasets, then testing SCISSORS against competing clustering methods.

## When to use

Use when comparing cell-type clustering results against retained ground-truth labels on simulated scRNA-seq data — varying cluster numbers, rare-cell proportions, and dropout — to quantify agreement via ARI, NMI, and silhouette scores.

## When NOT to use

- The dataset has no retained ground-truth cell-type labels — ARI and NMI require a known reference partition and cannot be computed from clusters alone
- The analysis goal is rare-cell subcluster discovery on real (unlabeled) data rather than benchmarking clustering accuracy on simulations — use SCISSORS's reclustering workflow directly instead
- You need to determine the optimal number of robust clusters rather than measure how well an existing partition recapitulates labels

## Inputs

- Simulated scRNA-seq count matrices derived from Zilionis 2019 human lung cancer and Baron 2016 human normal pancreas references, with retained ground-truth cell-type labels
- Seurat-processed scRNA-seq objects with PCA embeddings (NormalizeData → FindVariableFeatures vst nfeatures=2000 → ScaleData → RunPCA)
- Cluster label vectors from each method being compared (SCISSORS and seven baselines)

## Outputs

- Per-simulation ARI and NMI values for each clustering method versus ground truth
- Per-cell silhouette scores computed with CosineDist on PCA embeddings
- Method-by-metric results dataframe assembled with dplyr, with two-sided Wilcoxon rank-sum p-values vs SCISSORS
- Comparison figure panels (ARI/NMI bars, silhouette violin plots) rendered with ggplot2 and ggsignif; silhouette figure saved as Sil_Scores.pdf

## How to apply

Generate simulation datasets by subsampling and mixing reference cells (Zilionis 2019 human lung cancer and Baron 2016 human normal pancreas) across parameter combinations, retaining ground-truth labels for each simulation. Preprocess each simulation with the standard Seurat pipeline — NormalizeData, FindVariableFeatures (vst, 2000 features), ScaleData, RunPCA — then run SCISSORS in completely unsupervised mode with silhouette-score-driven reclustering alongside each competitor (Seurat/Louvain, GiniClust3, CellSIUS, k-means, hierarchical, DBSCAN, Leiden). Compute per-simulation ARI and NMI of each method's clusters against ground truth, and per-cell silhouette scores using cosine distance on the PCA embeddings (cluster::silhouette with CosineDist input). Assemble a method-by-metric dataframe with dplyr and test differences between SCISSORS and each competitor with two-sided Wilcoxon rank-sum tests. Render comparison panels (ARI/NMI bars, silhouette violin plots with geom_signif significance bars) in ggplot2.

## Related tools

- **SCISSORS** (Method under evaluation; run in completely unsupervised mode with silhouette-score-driven reclustering on each simulated dataset) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Standard preprocessing (NormalizeData, FindVariableFeatures with vst, ScaleData, RunPCA) and Louvain clustering baseline run on the same simulations)
- **giniclust3** (Competing baseline (Gini/Fano consensus clustering) run on identical simulated datasets for ARI/NMI comparison)
- **cluster** (Provides cluster::silhouette used to compute per-cell silhouette scores from CosineDist distance matrices)
- **dplyr** (Assembles the method-by-metric results dataframe of ARI, NMI, and silhouette values)
- **ggplot2** (Renders the Fig. 2a-c ARI/NMI bar panels and silhouette violin plots with ggsignif significance bars)

## Examples

```
cos_dist <- CosineDist(input = Embeddings(pbmc, "pca")); sils_SCISSORS <- ComputeSilhouetteScores(pbmc_SCISSORS, avg = FALSE)
```

## Evaluation signals

- SCISSORS must achieve the highest ARI and NMI against ground truth across simulations built from both the human lung cancer and human normal pancreas references
- SCISSORS must show the highest median silhouette score across all compared methods, with distributions visualized per cluster
- Differences between SCISSORS and each competitor must be statistically significant under two-sided Wilcoxon rank-sum tests, as reported in the paper
- Silhouette scores must be computed with cosine distance on the PCA embeddings, not Euclidean distance
- Each simulation must retain its ground-truth labels so per-simulation ARI/NMI can be recomputed reproducibly

## Limitations

- Even small differences in analysis and parameter choice can greatly alter scRNA-seq clustering results, so benchmark conclusions are sensitive to parameter settings
- Validation relies on simulated data derived from two reference datasets (Zilionis 2019 lung cancer, Baron 2016 pancreas); performance on other tissues or experimental designs is not established by this benchmark
- No discussion-section text was available in the provided document, so reproducibility gaps and further limitations could not be extracted

## Evidence

- [results] SCISSORS showed the best recapitulation of the clustering labels measured by ARI and NMI: [evidence span withheld — anchor status: not_found]
- [abstract] silhouette scoring for heterogeneity estimation: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process"
- [methods] ComputeSilhouetteScores call: "sils_SCISSORS <- ComputeSilhouetteScores(pbmc_SCISSORS, avg = FALSE)"
- [methods] CosineDist on PCA embeddings: "cos_dist <- CosineDist(input = Embeddings(pbmc, "pca"))"
- [methods] Seurat preprocessing parameters: "FindVariableFeatures(selection.method = "vst", nfeatures = 2000, verbose = FALSE)"
- [readme] package purpose from README: "This package enables researchers to efficiently, accurately, and reproducibly uncover rare celltypes and other small subgroups using an iterative reclustering optimization routine"
