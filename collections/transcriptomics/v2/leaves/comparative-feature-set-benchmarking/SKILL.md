---
name: comparative-feature-set-benchmarking
description: Use when a single-cell dataset has multiple candidate feature sets (e.g.,
  DUBStepR-selected peaks vs all peaks vs HVFs) and you need to decide which set recovers
  known biological structure such as a differentiation hierarchy.
when_to_use_negative:
- Input is already a curated feature table with a single fixed feature set, leaving
  nothing to compare
- The goal is to evaluate the DUBStepR algorithm itself on scRNA-seq gene expression
  rather than benchmark feature sets on scATAC-seq peaks
- No independently known biological structure (e.g., cell type hierarchy) exists to
  validate the recovered topology against
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DUBStepR
  role: Correlation-based feature selection run with optimise.features = TRUE, k =
    10, num.pcs = 20, error = 0 to select informative peaks
  repo: https://github.com/prabhakarlab/DUBStepR
- name: R
  role: Analysis environment in which the peak matrix is loaded, normalized, and all
    feature sets are benchmarked
- name: Seurat
  role: Recommended package for loading and log-normalizing the count matrix and building
    cell embeddings
- name: Monocle 3
  role: Trajectory inference run on each UMAP embedding to derive differentiation
    topologies for comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:24b1770a2dbe3357ee4536a09595eb24e575bd0d5b9d2e70f5c37d393fae5918
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/prabhakarlab/DUBStepR
  tools:
  - DUBStepR
  - Monocle 3
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# comparative-feature-set-benchmarking

## Summary

Benchmark a correlation-based feature selection method (DUBStepR) against comparator feature sets (all features, highly variable features) by running the same downstream pipeline (PCA → UMAP → Monocle 3) on each set and comparing the resulting cell-cluster/trajectory topologies.

## When to use

Use when a single-cell dataset has multiple candidate feature sets (e.g., DUBStepR-selected peaks vs all peaks vs HVFs) and you need to decide which set best recovers known biological structure such as a differentiation hierarchy.

## When NOT to use

- Input is already a curated feature table with a single fixed feature set, leaving nothing to compare
- The goal is to evaluate the DUBStepR algorithm itself on scRNA-seq gene expression rather than benchmark feature sets on scATAC-seq peaks
- No independently known biological structure (e.g., cell type hierarchy) exists to validate the recovered topology against

## Inputs

- Peak-by-cell scATAC-seq count matrix (e.g., Buenrostro et al. human bone marrow, GEO accession GSE96583)
- Comparator feature sets: all-peaks set and highly-variable-feature (HVF) peak set
- Known biological reference structure (e.g., hematopoietic differentiation hierarchy with lymphoid, myeloid, and megakaryocyte-erythroid branches)

## Outputs

- DUBStepR-selected informative peak set
- PCA + UMAP cell embeddings computed per feature set
- Monocle 3 trajectory topologies per feature set
- Comparison record stating which feature set recovers the known branching topology

## How to apply

Download the peak-by-cell count matrix (e.g., Buenrostro et al. human bone marrow scATAC-seq, GEO GSE96583) into R/Seurat, then log-normalize the peak accessibility matrix. Run DUBStepR with optimise.features = TRUE, k = 10, num.pcs = 20, error = 0, and min.cells set to 5% of cells, adapting the gene-based vignette call to peaks; independently define the all-peaks and HVF comparator feature sets. For each of the three feature sets, run PCA followed by UMAP embedding of the cells, then run Monocle 3 trajectory inference on each UMAP embedding. Compare the three inferred topologies and record which feature set reproduces the known lymphoid/myeloid/megakaryocyte-erythroid branching structure. The rationale is that existing feature selection methods ignore gene-gene correlation information and perform inconsistently across datasets, so only a head-to-head comparison on the same downstream pipeline reveals which feature set genuinely improves cell type inference.

## Related tools

- **DUBStepR** (Correlation-based feature selection run with optimise.features = TRUE, k = 10, num.pcs = 20, error = 0 to select informative peaks) — https://github.com/prabhakarlab/DUBStepR
- **R** (Analysis environment in which the peak matrix is loaded, normalized, and all feature sets are benchmarked)
- **Seurat** (Recommended package for loading and log-normalizing the count matrix and building cell embeddings)
- **Monocle 3** (Trajectory inference run on each UMAP embedding to derive differentiation topologies for comparison)

## Examples

```
selected.peaks <- DUBStepR(input.data = lognorm.peaks, min.cells = ncol(lognorm.peaks)*0.05, species = "Homo_sapiens", optimise.features = TRUE, k = 10, num.pcs = 20, error = 0)
```

## Evaluation signals

- The DUBStepR-selected peak set's Monocle 3 trajectory recovers the three-branch lymphoid/myeloid/megakaryocyte-erythroid topology while the all-peaks and HVF sets fail to do so (matching Fig 7)
- DUBStepR is run with the specified parameters (optimise.features = TRUE, k = 10, num.pcs = 20, error = 0, min.cells = 5% of cells) so results are reproducible
- Each of the three feature sets passes through an identical PCA → UMAP → Monocle 3 pipeline, ensuring differences come from the feature sets alone
- The selected feature subset is relatively small (DUBStepR approaches its maximum silhouette index at 200–500 feature genes), so a DUBStepR set comparable in size to full data indicates misconfiguration

## Limitations

- Existing feature selection methods perform inconsistently across datasets and can yield worse clustering accuracy than no feature selection, so a winning feature set on one dataset may not transfer
- The DUBStepR vignette is gene-based; applying it to peaks is an adaptation not covered by the original documentation
- Elbow point computation and key benchmarking details are deferred to supplementary notes and figures not included in the main text

## Evidence

- [abstract] DUBStepR scales to over a million cells and can be applied to other data types such as single-cell ATAC-seq: "DUBStepR is scalable to over a million cells, and can be straightforwardly applied to other data types such as single-cell ATAC-seq."
- [abstract] Existing feature selection methods perform inconsistently across datasets and can yield worse clustering accuracy than no feature selection: "Existing feature selection methods perform inconsistently across datasets, occasionally even resulting in poorer clustering accuracy than without feature selection."
- [methods] DUBStepR function signature and parameters: "DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error)"
- [readme] Correlation-based rationale of DUBStepR feature selection: "DUBStepR is based on the intuition that cell-type-specific marker genes tend to be well correlated with each other"
- [discussion] Feature-set size guidance for clustering performance: "Intriguingly, DUBStepR approaches its maximum silhouette index value at 200–500 feature genes (Supp. Fig. S3)"
