---
name: scissors-derived-marker-gene-selection
description: Use when a heterogeneous Seurat cluster needs rare subclusters resolved
  and a specific marker-gene signature, or when such a signature must be applied de
  novo to score bulk RNA-seq samples into subtypes.
when_to_use_negative:
- When clusters are already homogeneous (high median silhouette scores) and no rare
  subgroups are suspected — reclustering adds noise rather than resolving subpopulations
- When only the optimal number of robust clusters is needed for abundant cell types,
  not rare-cell resolution or marker-gene derivation
- When bulk data cannot be anchored to a reference schema (e.g., Moffitt genes), since
  signature-based bulk calls need a validity anchor
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCISSORS
  role: 'Core framework: silhouette scoring, semi-supervised iterative reclustering,
    and marker-gene identification'
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Single-cell infrastructure wrapping the SCISSORS workflow (NormalizeData,
    FindVariableFeatures, ScaleData, RunPCA, FindNeighbors)
  repo: https://github.com/jr-leary7/SCISSORS
- name: dplyr
  role: Tidy data manipulation within the R workflow
- name: cluster
  role: Clustering tools supporting silhouette estimation
- name: ggplot2
  role: Visualization of silhouette score distributions and subcluster plots
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:050d4c5febbf0cf2e135b8fbeb4986d6159bfa0b5ece2ec921a1fefb7419699b
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# SCISSORS-derived marker gene selection

## Summary

Derives highly specific marker-gene signatures from SCISSORS-identified single-cell subclusters (built on Seurat preprocessing and silhouette-scored reclustering), then applies the fold-change-ranked signatures as scoring sets on external data such as TCGA bulk RNA-seq subtyping.

## When to use

Use when a heterogeneous Seurat cluster needs rare subclusters resolved and a specific marker-gene signature, or when such a signature must be applied de novo to score bulk RNA-seq samples into subtypes.

## When NOT to use

- When clusters are already homogeneous (high median silhouette scores) and no rare subgroups are suspected — reclustering adds noise rather than resolving subpopulations
- When only the optimal number of robust clusters is needed for abundant cell types, not rare-cell resolution or marker-gene derivation
- When bulk data cannot be anchored to a reference schema (e.g., Moffitt genes), since signature-based bulk calls need a validity anchor

## Inputs

- Single-cell RNA-seq count matrix / Seurat object (e.g., PBMC3k from SeuratData)
- Broad-cluster Seurat object with UMAP embedding and cluster assignments
- TCGA PAAD bulk RNA-seq expression matrix (150 tumors) for external application of derived signatures

## Outputs

- Per-cell silhouette scores for each broad cluster
- Reclustered rare subclusters and their annotations
- High-specificity marker-gene sets per subcluster, fold-change ranked (e.g., top-10 basal-like and classical genes)
- Per-sample subtype assignments on bulk data (e.g., 33 basal-like / 106 classical PAAD calls) with concordance and survival statistics

## How to apply

Preprocess the scRNA-seq object with the SCISSORS-wrapped Seurat pipeline: NormalizeData, FindVariableFeatures (vst, nfeatures = 2000), ScaleData, RunPCA, and FindNeighbors on dims 1:10, then PrepareData with chosen n.HVG, n.PC, and initial.resolution for broad Louvain clustering. Estimate per-cell silhouette scores with ComputeSilhouetteScores to flag heterogeneous broad clusters, then run the multi-step semi-supervised reclustering (e.g., getClusters at the optimal k) to resolve rare subclusters. Identify marker genes highly specific to each subcluster and rank them by fold change; in the TCGA PAAD application the top-10 fold-change-ranked basal-like and classical genes (anchored by overlap with Moffitt genes) are used. Score each bulk tumor against both signatures and assign the subtype with the higher score; validate concordance against published schema calls and test survival separation by log-rank.

## Related tools

- **SCISSORS** (Core framework: silhouette scoring, semi-supervised iterative reclustering, and marker-gene identification) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Single-cell infrastructure wrapping the SCISSORS workflow (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors)) — https://github.com/jr-leary7/SCISSORS
- **dplyr** (Tidy data manipulation within the R workflow)
- **cluster** (Clustering tools supporting silhouette estimation)
- **ggplot2** (Visualization of silhouette score distributions and subcluster plots)

## Examples

```
sil_score_df <- ComputeSilhouetteScores(seu_pbmc, avg = FALSE)
```

## Evaluation signals

- Silhouette score distributions for heterogeneous clusters should show distinctly lower medians than homogeneous clusters before reclustering
- Derived signature calls on TCGA PAAD should reproduce 139/150 concordance with Moffitt schema calls (33 basal-like, 106 classical)
- Log-rank test of basal-like vs. classical survival should yield P = .025 with hazard ratio = 1.723 (shorter basal-like survival)
- Marker genes should show high specificity to their subcluster and anchor to known reference genes (Moffitt basal-like/classical overlap)
- Subcluster counts should be reproducible with a fixed random.seed in PrepareData (e.g., seed = 629, initial.resolution = 0.4 gives 6 unique clusters on PBMC3k)

## Limitations

- Small differences in analysis and parameter choice can greatly alter scRNA-seq clustering results, affecting which cell types are identified
- Existing methods focus on the optimal number of robust clusters, which is problematic for extremely low-abundance cells due to their subtle contributions
- No discussion-section text was available in the source document, so reproducibility gaps and limitations could not be extracted from the discussion section

## Evidence

- [abstract] SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised recl"
- [abstract] Additionally, SCISSORS provides a method for the identification of marker genes of high specificity to the cell type: "Additionally, SCISSORS provides a method for the identification of marker genes of high specificity to the cell type"
- [methods] NormalizeData followed by FindVariableFeatures with vst and 2000 features: "NormalizeData(verbose = FALSE) %>% 
               FindVariableFeatures(selection.method = "vst", nfeatures = 2000, verbose = FALSE)"
- [readme] README statement that SCISSORS is an iterative reclustering optimization routine extending a typical Seurat workflow: "uncover rare celltypes and other small subgroups using an
iterative reclustering optimization routine built as an extension to a
typical `Seurat` workflow"
