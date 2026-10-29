---
name: cite-seq-batch-correction-comparison
description: Use when combining two or more CITE-seq datasets (e.g., pbmc_1k_protein_v3
  and pbmc_10k_protein_v3) and you need to decide whether Seurat batch correction
  changes the cell groups identified by multimodal LinQ-View clustering.
when_to_use_negative:
- Datasets with a very high number of ADT features (e.g., 170K cells with 228 ADT
  features) — the article notes LinQ-View failed on such massive datasets while Seurat
  4 succeeded
- Large datasets where LinQ-View's quadratic pairwise-distance computation makes it
  too time-inefficient — the article recommends Seurat 4 as the best option for massive
  CITE-seq datasets with many ADT features
- Single-dataset analyses with no batch structure to compare against — a batch-correction
  vs no-correction comparison is meaningless without at least two combined sources
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: LinQ-View
  role: CITE-seq toolkit used for multimodal pre-processing, sigmoid-filtered ADT
    distance calculation, L∞ norm joint-distance fusion, and Louvain clustering of
    corrected vs uncorrected runs
  repo: https://github.com/WilsonImmunologyLab/LinQView
- name: Seurat v3.9.9
  role: Provides the batch-effect correction applied to the combined PBMC 1K + PBMC
    10K data before joint-distance fusion and clustering
  repo: https://satijalab.org/seurat/
- name: R v3.6.3
  role: Runtime environment in which LinQ-View and all pre-processing, clustering,
    and visualization steps are executed
  repo: https://www.r-project.org/
- name: umap v0.2.6.0
  role: Non-linear dimension reduction used to generate UMAP embeddings for visual
    comparison of both clustering runs
  repo: https://github.com/lmcinnes/umap
- name: ggplot2 v3.3.2
  role: Renders the side-by-side cluster-comparison figure panel from the two runs
  repo: https://dplyr.tidyverse.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100056
    title: Improved integration of single-cell transcriptome and surface protein expression
      by LinQ-View
  source_content_hash: sha256:22bcb172937430e5f3ef95dcceb1aa669f94f0ee26ff5d6de34f6381de6b49be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WilsonImmunologyLab/LinQView
  tools:
  - LinQ-View
  - R
  - R v3.6.3
  - Seurat
  - Seurat v3.9.9
  - ggplot2
  - ggplot2 v3.3.2
  - umap v0.2.6.0
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cite-seq-batch-correction-comparison

## Summary

A benchmarking skill for quantifying how Seurat 3 batch-effect correction changes the number of cell groups LinQ-View identifies from combined CITE-seq datasets, by comparing a batch-corrected run against a no-correction negative control. Rationale: batch effects can split or merge clusters, so cluster-count deltas (here 21 vs 24 groups on PBMC 1K + PBMC 10K) reveal which groups are batch-driven artifacts.

## When to use

Use when combining two or more CITE-seq datasets (e.g., pbmc_1k_protein_v3 and pbmc_10k_protein_v3) and you need to decide whether Seurat batch correction changes the cell groups identified by multimodal LinQ-View clustering.

## When NOT to use

- Datasets with a very high number of ADT features (e.g., 170K cells with 228 ADT features) — the article notes LinQ-View failed on such massive datasets while Seurat 4 succeeded
- Large datasets where LinQ-View's quadratic pairwise-distance computation makes it too time-inefficient — the article recommends Seurat 4 as the best option for massive CITE-seq datasets with many ADT features
- Single-dataset analyses with no batch structure to compare against — a batch-correction vs no-correction comparison is meaningless without at least two combined sources

## Inputs

- 10x Genomics pbmc_1k_protein_v3 CITE-seq dataset (gene-expression + ADT counts)
- 10x Genomics pbmc_10k_protein_v3 CITE-seq dataset (gene-expression + ADT counts)
- LinQ-View v0.99 analysis environment in R v3.6.3
- Seurat v3.9.9 batch-correction method

## Outputs

- Cell-group count table comparing batch-corrected (21 groups) vs negative-control (24 groups) runs
- UMAP embeddings (umap v0.2.6.0) for the corrected and uncorrected runs
- Comparison figure panel (rendered with ggplot2 v3.3.2, e.g., Figure 5)
- List of batch-affected cluster identities (e.g., clusters 11, 12, 16, 22 in the negative control)

## How to apply

In R v3.6.3 with LinQ-View v0.99, download the pbmc_1k_protein_v3 and pbmc_10k_protein_v3 datasets from 10x Genomics and pre-process each: exclude cells with fewer than 200 or more than 2500 detected genes, apply a mitochondrial-gene soft threshold at the 95th percentile with a 10% sealing point, then normalize and scale both transcriptome and ADT modalities, identify HVGs, and run PCA. For the negative control, compute cell-cell pairwise distances per modality (applying the sigmoid filter to the ADT distance), fuse modalities with the L∞ norm joint distance, and cluster with the default Seurat community-detection (Louvain) algorithm without batch correction. For the corrected run, combine PBMC 1K + PBMC 10K and apply Seurat v3.9.9 batch correction before repeating the same joint-distance fusion and clustering. Count cell groups in each run (expected: 24 groups uncorrected with batch-affected clusters flagged, 21 groups corrected), generate UMAP embeddings (umap v0.2.6.0) for both runs, and render the side-by-side comparison with ggplot2 v3.3.2.

## Related tools

- **LinQ-View** (CITE-seq toolkit used for multimodal pre-processing, sigmoid-filtered ADT distance calculation, L∞ norm joint-distance fusion, and Louvain clustering of corrected vs uncorrected runs) — https://github.com/WilsonImmunologyLab/LinQView
- **Seurat v3.9.9** (Provides the batch-effect correction applied to the combined PBMC 1K + PBMC 10K data before joint-distance fusion and clustering) — https://satijalab.org/seurat/
- **R v3.6.3** (Runtime environment in which LinQ-View and all pre-processing, clustering, and visualization steps are executed) — https://www.r-project.org/
- **umap v0.2.6.0** (Non-linear dimension reduction used to generate UMAP embeddings for visual comparison of both clustering runs) — https://github.com/lmcinnes/umap
- **ggplot2 v3.3.2** (Renders the side-by-side cluster-comparison figure panel from the two runs) — https://dplyr.tidyverse.org/

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("WilsonImmunologyLab/LinQView")
```

## Evaluation signals

- Negative-control run (no batch correction) on combined PBMC 1K + PBMC 10K yields exactly 24 cell groups under default LinQ-View Louvain clustering
- Seurat 3 batch-corrected run on the same combined data yields exactly 21 cell groups
- Batch-affected clusters in the negative control (clusters 11, 12, 16, 22) are resolved/merged after correction, consistent with a cluster-count delta of 3
- Both runs use identical pre-processing thresholds: 200–2500 detected gene cutoffs and a mitochondrial soft threshold at the 95th percentile with a 10% sealing point
- UMAP embeddings for both runs are generated with umap v0.2.6.0 and the figure panel renders successfully with ggplot2 v3.3.2

## Limitations

- LinQ-View's pairwise distance computation scales quadratically with cell number, limiting time efficiency on large combined datasets
- LinQ-View failed on a massive dataset of 170K cells with 228 ADT features, whereas Seurat 4 succeeded; Seurat 4 is recommended for massive CITE-seq datasets with many ADT features
- LinQ-View is best suited for large datasets with a limited number of ADT features (e.g., <50)

## Evidence

- [results] LinQ-View analysis with Seurat 3 batch correction identified 21 cell groups: [evidence span withheld — anchor status: not_found]
- [methods] Cell QC thresholds: "all cells that have less than 200 genes detected or more than 2500 genes detected have been excluded"
- [methods] Mitochondrial soft threshold: "A soft threshold was set to the 95th percentile of the current dataset distribution, and the soft threshold was subject to a sealing point as the maximum threshold"
- [intro] Sigmoid filter and L∞ fusion: "LinQ-View integrates information from transcriptome and cell-surface protein expression levels by fusing pairwise distance matrices with an L∞ norm"
- [methods] Default clustering method: "The default cell population clustering method in LinQ-View is a community detection algorithm implemented in Seurat"
- [discussion] Dataset availability: "Data is available from 10x Genomics https://support.10xgenomics.com/single-cell-gene-expression/datasets/3.0.0/pbmc_1k_protein_v3"
- [readme] README primary idea: "The primary idea behind LinQ-View for CITE-seq is to calculate the dissimilarity representations for each single cell profiling modality (i.e. mRNA and surface protein expression) and then combine"
