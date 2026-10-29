---
name: unsupervised-scrna-seq-clustering-reclustering
description: Use when the input is a raw scRNA-seq count matrix (e.g. 10X PBMC3k)
  whose initial unsupervised clusters appear heterogeneous, and rare or low-abundance
  cell populations must be resolved without prior labels.
when_to_use_negative:
- The goal is copy-number variation inference from scRNA-seq count tables or BAM files
  — that is the scope of CONICS/CONICSmat, not silhouette-driven reclustering.
- The dataset already has reliable fine-grained cell-type labels or the clusters show
  high median silhouette scores, so no heterogeneous target clusters remain for reclustering.
- The task is purely 2D embedding/visualization of a large transcriptome matrix (e.g.
  t-SNE via openTSNE) rather than partitioning cells into clusters.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCISSORS
  role: 'Core framework: silhouette-score-driven iterative reclustering of heterogeneous
    Seurat clusters and marker-gene identification'
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Single-cell infrastructure for preprocessing (NormalizeData, FindVariableFeatures
    vst, ScaleData, RunPCA, FindNeighbors) and initial Louvain clustering
- name: cluster
  role: R package supplying silhouette-score computation for estimating cluster heterogeneity
- name: CosineDist
  role: Computes the cosine distance matrix from PCA embeddings used as input to silhouette
    scoring
- name: ggplot2
  role: Renders silhouette-score distribution plots per broad cluster to select reclustering
    targets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:f1d788718f457062650323b9ea28cb33541c4a5d796d4153030cb3a2389870c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jr-leary7/SCISSORS
  tools:
  - CosineDist
  - R
  - SCISSORS
  - Seurat
  - cluster
  - dplyr
  - ggplot2
  - ggsignif
  - giniclust3
  - magrittr
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# unsupervised-scrna-seq-clustering-reclustering

## Summary

Unsupervised clustering and silhouette-driven reclustering of scRNA-seq data using SCISSORS, a framework wrapped around Seurat, to uncover rare cell types and small subgroups within heterogeneous broad clusters.

## When to use

Use when the input is a raw scRNA-seq count matrix (e.g. 10X PBMC3k) whose initial unsupervised clusters appear heterogeneous, and rare or low-abundance cell populations must be resolved without prior labels.

## When NOT to use

- The goal is copy-number variation inference from scRNA-seq count tables or BAM files — that is the scope of CONICS/CONICSmat, not silhouette-driven reclustering.
- The dataset already has reliable fine-grained cell-type labels or the clusters show high median silhouette scores, so no heterogeneous target clusters remain for reclustering.
- The task is purely 2D embedding/visualization of a large transcriptome matrix (e.g. t-SNE via openTSNE) rather than partitioning cells into clusters.

## Inputs

- Raw scRNA-seq count matrix (e.g. 10X PBMC3k loaded via SeuratData::LoadData('pbmc3k'))
- Seurat object after initial preprocessing (normalized, HVG-selected, PCA-embedded)
- PCA embeddings for silhouette computation (Embeddings(pbmc, 'pca'))
- Ground-truth cell-type labels when evaluating on simulated data (e.g. Zilionis 2019 lung cancer, Baron 2016 pancreas references)

## Outputs

- Broad and reclustered cluster assignments per cell (getClusters output, e.g. optK = 3)
- Per-cell silhouette score dataframe from ComputeSilhouetteScores for violin-plot inspection
- UMAP embedding of broad clusters (DimPlot of Louvain clusters)
- Cluster-specific marker genes for annotating subclusters

## How to apply

Preprocess the count matrix with the standard Seurat pipeline — NormalizeData, FindVariableFeatures (vst, nfeatures = 2000), ScaleData, RunPCA, FindNeighbors (dims = 1:10) — or use the SCISSORS PrepareData wrapper (e.g. n.HVG = 4000, n.PC = 15, which.dim.reduc = 'umap', initial.resolution = 0.4, random.seed = 629) to obtain broad Louvain clusters and a UMAP embedding. Estimate per-cell silhouette scores with ComputeSilhouetteScores(avg = FALSE) using cosine distance (CosineDist) computed on the PCA embeddings, and visualize the per-cluster distributions to flag clusters with low median silhouette (e.g. clusters 0, 1, and 3 in the PBMC3k example). Recluster only the flagged heterogeneous clusters via SCISSORS's multi-step semi-supervised reclustering routine, running in completely unsupervised mode with silhouette-score-driven reclustering when no marker knowledge is available. Finally, examine cluster-specific genes to annotate the resulting subclusters; SCISSORS additionally provides a method for identifying highly specific marker genes.

## Related tools

- **SCISSORS** (Core framework: silhouette-score-driven iterative reclustering of heterogeneous Seurat clusters and marker-gene identification) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Single-cell infrastructure for preprocessing (NormalizeData, FindVariableFeatures vst, ScaleData, RunPCA, FindNeighbors) and initial Louvain clustering)
- **cluster** (R package supplying silhouette-score computation for estimating cluster heterogeneity)
- **CosineDist** (Computes the cosine distance matrix from PCA embeddings used as input to silhouette scoring)
- **ggplot2** (Renders silhouette-score distribution plots per broad cluster to select reclustering targets)

## Examples

```
seu_pbmc <- SeuratData::LoadData("pbmc3k") %>% PrepareData(n.HVG = 4000, n.PC = 15, which.dim.reduc = "umap", initial.resolution = 0.4, use.parallel = FALSE, random.seed = 629)
```

## Evaluation signals

- On PBMC3k, PrepareData reports 'Found 6 unique clusters' at initial.resolution = 0.4; a markedly different cluster count signals a parameter or data-loading problem.
- Silhouette-score distributions must be computed per broad cluster, and flagged targets (e.g. clusters 0, 1, 3 with lower median scores) must correspond to visually heterogeneous clusters on the UMAP.
- On simulated benchmarks (Zilionis 2019 lung cancer / Baron 2016 pancreas), SCISSORS should achieve the highest ARI and NMI against ground truth versus Seurat/Louvain, GiniClust3, CellSIUS, k-means, hierarchical, DBSCAN, and Leiden.
- Per-cell silhouette scores must be computed with cosine distance on PCA embeddings (cluster::silhouette with CosineDist input); using Euclidean distance on scaled data instead deviates from the method.
- SCISSORS must show the highest median silhouette score across methods, with differences versus competitors significant under two-sided Wilcoxon rank-sum tests.

## Limitations

- Existing methods largely focus on determining the optimal number of robust clusters, which is problematic for identifying cells of extremely low abundance due to their subtle contributions toward clustering metrics.
- Even small differences in analysis and parameter choice can greatly alter clustering results and thus which cell types are identified; seeds and resolution settings (e.g. random.seed = 629, initial.resolution = 0.4) should be recorded for reproducibility.
- No discussion-section limitations or changelog were available in the provided document for this skill's provenance, so additional caveats could not be extracted.

## Evidence

- [abstract] SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process"
- [methods] NormalizeData / FindVariableFeatures vst 2000 / ScaleData / RunPCA pipeline quote: "NormalizeData(verbose = FALSE) %>% 
               FindVariableFeatures(selection.method = "vst", nfeatures = 2000, verbose = FALSE)"
- [methods] CosineDist on PCA embeddings: "cos_dist <- CosineDist(input = Embeddings(pbmc, "pca"))"
- [methods] ComputeSilhouetteScores per cell: "sils_SCISSORS <- ComputeSilhouetteScores(pbmc_SCISSORS, avg = FALSE)"
- [readme] README PrepareData invocation on PBMC3k: "seu_pbmc <- SeuratData::LoadData("pbmc3k") %>% 
            PrepareData(n.HVG = 4000, 
                        n.PC = 15, 
                        which.dim.reduc = "umap""
- [readme] README low-silhouette clusters flagged for reclustering: [evidence span withheld — anchor status: external_doc]
