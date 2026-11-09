---
name: nngraph-clustering-of-cell-embeddings
description: Use when you have a cell-by-dimension embedding matrix (e.g., CA embeddings
  from corral on counts, or PCA on log(x+1) logcounts) and need to assign cells to
  clusters to quantify recovery of annotated cell types by ARI.
when_to_use_negative:
- You need de novo discovery of biologically novel sub-populations or rare cell types
  — ARI against a single 'reference' labeling can penalize methods that better distinguish
  sub-populations
- Your dataset is a complex tissue whose cell identities were assigned by a single
  analytical method without independent validation — benchmark quality is limited
  in that setting
- The number of true clusters is expected to differ from the reference annotations'
  a priori cluster count, which is unknown and context-dependent
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: corral
  role: Computes the CA cell embeddings (via SVD of Pearson chi-squared residuals)
    on which NNGraph clustering is performed
  repo: github.com/laurenhsu1/corral
- name: R
  role: Implementation environment for the clustering and ARI evaluation workflow
- name: Bioconductor
  role: Ecosystem providing SingleCellExperiment classes that corral interfaces with
    directly
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:681b271e96fef2e5d1303dfb87b0b4d50a265bf66c618ece112d483565abff93
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - corral
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# NNGraph clustering of cell embeddings

## Summary

Clusters single-cell RNA-seq cells in a reduced-dimension embedding (from CA via corral or PCA on logcounts) using NNGraph clustering, then scores cluster recovery against published ground-truth cell-type labels with Adjusted Rand Index, as used to benchmark CA versus PCA on SCMixology and Zhengmix datasets.

## When to use

Use when you have a cell-by-dimension embedding matrix (e.g., CA embeddings from corral on counts, or PCA on log(x+1) logcounts) and need to assign cells to clusters to quantify recovery of annotated cell types by ARI.

## When NOT to use

- You need de novo discovery of biologically novel sub-populations or rare cell types — ARI against a single 'reference' labeling can penalize methods that better distinguish sub-populations
- Your dataset is a complex tissue whose cell identities were assigned by a single analytical method without independent validation — benchmark quality is limited in that setting
- The number of true clusters is expected to differ from the reference annotations' a priori cluster count, which is unknown and context-dependent

## Inputs

- Reduced-dimension cell embedding matrix (cells x dimensions), e.g. CA embeddings from corral on a raw counts matrix or PCA embeddings of logcounts
- Raw scRNA-seq counts matrix (genes x cells) with SingleCellExperiment-compatible structure
- Ground-truth cell-type annotation vector per cell (from original dataset publications, e.g. SCMixology / Zhengmix labels)

## Outputs

- Per-cell cluster assignments from NNGraph clustering of the embedding
- Adjusted Rand Index score of clusters vs annotated cell types, per dataset and per dimension-reduction method
- Comparison table of ARI values (e.g., ari_comparison.csv) across CA and PCA pipelines

## How to apply

Take the reduced-dimension embeddings produced by dimension reduction — CA computed by SVD of the Pearson chi-squared residuals via the corral R/Bioconductor package on raw counts, or PCA on the log(x + 1) transformed counts (logcounts) — and run NNGraph clustering on the cell coordinates. Apply this per dataset; in the source benchmark the inputs were three SCMixology cell-mixture datasets (CellBench) and three Zhengmix datasets (DuoClustering2018), each with published ground-truth labels. Compute the Adjusted Rand Index of the resulting cluster assignments against the annotated cell types, and compile ARI values across methods (CA vs PCA) and datasets (e.g., into ari_comparison.csv) for comparison. A successful run yields clusters whose ARI against the reference labels can be directly compared between dimension reduction strategies on identical input data.

## Related tools

- **corral** (Computes the CA cell embeddings (via SVD of Pearson chi-squared residuals) on which NNGraph clustering is performed) — github.com/laurenhsu1/corral
- **R** (Implementation environment for the clustering and ARI evaluation workflow)
- **Bioconductor** (Ecosystem providing SingleCellExperiment classes that corral interfaces with directly)

## Evaluation signals

- Adjusted Rand Index is computed against the published ground-truth cell-type labels for every dataset, not just visually inspected
- ARI values are comparable only when NNGraph clustering is run on embeddings from the same dataset for both CA (counts) and PCA (logcounts) pipelines
- ARI lies in a bounded range (roughly 0 to 1 for agreement with reference labels); near-0 values indicate label-shuffled clustering
- Benchmark expectation from the article: CA outperformed or matched PCA in every counts/logcounts comparison across the three SCMixology and three Zhengmix datasets
- All six benchmark datasets (three SCMixology, three Zhengmix) must be processed with identical clustering settings for the comparison table to be valid

## Limitations

- Benchmark ground-truth labels come from original dataset publications and were not independently validated, so performance is limited by the original cell type assignments
- ARI evaluation may penalize methods that better distinguish sub-populations or rare cell types relative to reference labels
- High-quality ground-truth benchmarking datasets (SCMixology, Zhengmix) are simple pseudo-cell mixtures or pools of distinct cell types and do not reflect real-world tissue complexity
- No tool or package versions for corral or dependencies are reported in the article
- The number of 'true' clusters is unknown a priori and depends on the specific research question and context

## Evidence

- [intro] CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation: "CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation"
- [intro] reduced dimension embeddings used for clustering: "the resulting reduced dimension embeddings of the data from PCA are nonetheless used in scRNAseq clustering, trajectory analysis, and cell type classification"
- [abstract] CA-FT most performant in cluster recovery: "CA with Freeman-Tukey residuals performs especially well across diverse datasets"
- [discussion] ground-truth labels not independently validated: "the datasets we analyzed did not have independently validated cell type annotations, so performance is limited by the original cell type assignments"
- [readme] corral package purpose: "corral is a package to perform correspondence analysis on count data, particularly for single-cell (e.g., RNA-seq)."
