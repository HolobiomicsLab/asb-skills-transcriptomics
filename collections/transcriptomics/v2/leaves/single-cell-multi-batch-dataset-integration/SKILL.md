---
name: single-cell-multi-batch-dataset-integration
description: Use when concatenating multiple scRNA-seq count matrices (e.g., Baron,
  Lawlor, Muraro pancreas datasets) with batch and cell-type annotations and you need
  joint low-dimensional embeddings that both preserve cell-type separation and integrate
  batches for comparison against PCA-based pipelines.
when_to_use_negative:
- Data are simple pseudo-cell mixtures or pools of distinct cell types (e.g., SCMixology,
  Zhengmix) where batch structure is by design simpler than real complex tissue —
  the paper notes these do not reflect real-world tissue complexity
- A single-table dimensionality reduction is needed without multi-batch alignment
  — use the corral function instead of corralm
- Cell-type ground-truth labels are unvalidated or the goal is to detect novel sub-populations/rare
  cell types — the paper notes ARI-style evaluation can penalize methods that better
  distinguish sub-populations relative to reference labels
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: corralm
  role: Multi-table CA-based alignment and batch integration of multiple scRNA-seq
    count tables, using Freeman–Tukey residuals
  repo: github.com/laurenhsu1/corral
- name: corral
  role: Single-table correspondence-analysis dimension reduction; corralm is its multi-table
    counterpart within the same R/Bioconductor package
  repo: github.com/laurenhsu1/corral
- name: Harmony
  role: Batch correction applied to corralm embeddings to form the corralm+Harmony
    integration variant
- name: MNN
  role: Batch correction applied to corralm embeddings to form the corralm+MNN integration
    variant
- name: Bioconductor
  role: Ecosystem providing SingleCellExperiment classes that corral/corralm interface
    with directly
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:83c7e528711786a5ea696641ed61c3b0e56e260315deb5e8083fa85f591eedf0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - Harmony
  - MNN
  - R
  - corral
  - corralm
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# single-cell-multi-batch-dataset-integration

## Summary

Integrate multiple single-cell RNA-seq batches/datasets by running corralm multi-table correspondence analysis (preferably with Freeman–Tukey residuals) on concatenated count matrices, optionally followed by batch correction (Harmony or MNN) on the resulting cell embeddings, to preserve cell-type structure while removing batch effects.

## When to use

Use when concatenating multiple scRNA-seq count matrices (e.g., Baron, Lawlor, Muraro pancreas datasets) with batch and cell-type annotations and you need joint low-dimensional embeddings that both preserve cell-type separation and integrate batches for comparison against PCA-based pipelines.

## When NOT to use

- Data are simple pseudo-cell mixtures or pools of distinct cell types (e.g., SCMixology, Zhengmix) where batch structure is by design simpler than real complex tissue — the paper notes these do not reflect real-world tissue complexity
- A single-table dimensionality reduction is needed without multi-batch alignment — use the corral function instead of corralm
- Cell-type ground-truth labels are unvalidated or the goal is to detect novel sub-populations/rare cell types — the paper notes ARI-style evaluation can penalize methods that better distinguish sub-populations relative to reference labels

## Inputs

- Multiple scRNA-seq count matrices (e.g., Baron, Lawlor, Muraro pancreas datasets) as SingleCellExperiment objects
- Per-cell cell-type annotations
- Per-cell batch/sample labels

## Outputs

- Integrated low-dimensional cell embeddings from corralm (and corralm+Harmony / corralm+MNN variants)
- Per-method ASW_celltype and 1-ASWbatch comparison table (e.g., asw_comparison.csv)
- Batch-corrected embeddings from Harmony/MNN applied to corralm output

## How to apply

Load each dataset's counts into SingleCellExperiment objects in R/Bioconductor, retaining cell type and batch annotations, and concatenate them. Run corralm on the multi-table data using Freeman–Tukey residuals (CA-FT, the adaptation the paper found most performant) to obtain integrated cell embeddings that avoid distortive log-transformation. To emulate integration pipelines, apply Harmony or MNN batch correction downstream on the corralm embeddings, and run multibatch PCA as the non-CA baseline. Evaluate on the first 8 dimensions of each method's embedding by computing average silhouette width with cell-type labels (ASW_celltype) and with batch labels reported as 1-ASWbatch, tabulating both per method (e.g., saving asw_comparison.csv).

## Related tools

- **corralm** (Multi-table CA-based alignment and batch integration of multiple scRNA-seq count tables, using Freeman–Tukey residuals) — github.com/laurenhsu1/corral
- **corral** (Single-table correspondence-analysis dimension reduction; corralm is its multi-table counterpart within the same R/Bioconductor package) — github.com/laurenhsu1/corral
- **Harmony** (Batch correction applied to corralm embeddings to form the corralm+Harmony integration variant)
- **MNN** (Batch correction applied to corralm embeddings to form the corralm+MNN integration variant)
- **Bioconductor** (Ecosystem providing SingleCellExperiment classes that corral/corralm interface with directly)

## Examples

```
sce <- SingleCellExperiment(assays=list(counts=cbind(Baron, Lawlor, Muraro))); sce <- corralm(sce, whichmat='counts', method='freeman-tukey'); sce <- harmony::RunHarmony(sce, group.by.vars='batch', embedding.use='corralm')
```

## Evaluation signals

- ASW_celltype computed on the first 8 embedding dimensions is comparable to or better than the multibatch PCA baseline (corralm+Harmony matched Seurat's routine performance in the pancreas benchmark)
- 1-ASWbatch on the first 8 dimensions indicates batch mixing comparable to established integration methods (e.g., Seurat)
- Embeddings are produced without a distortive log(x+1) transformation — the CA is computed on a chi-squared (Freeman–Tukey) residual matrix of raw counts
- The comparison table (asw_comparison.csv) contains both ASW_celltype and 1-ASWbatch rows per method, computed on the same number of dimensions (8) across methods
- corralm runs directly on SingleCellExperiment objects without conversion out of Bioconductor classes

## Limitations

- No tool or package versions reported for corral or its dependencies
- Benchmark ground-truth labels come from original dataset publications and were not independently validated, limiting performance assessment
- ARI-style evaluation may penalize methods that better distinguish sub-populations or rare cell types relative to reference labels
- High-quality ground-truth benchmarking datasets (SCMixology, Zhengmix) are limited to simple pseudo-cell mixtures and do not reflect real-world tissue complexity; no accession IDs are provided for them

## Evidence

- [abstract] corralm for integrative multi-table dimension reduction of scRNAseq data: "we introduce corralm for integrative multi-table dimension reduction of scRNAseq data"
- [discussion] CA-FT most performant: "CA-FT was most performant overall in a scRNAseq cluster recovery task."
- [readme] package usage: "1. `corral` for dimensionality reduction on a single table 
2. `corralm` for alignment and batch integration of multiple tables"
- [abstract] pipeline substitution: "Switching from PCA to CA is achieved through a simple pipeline substitution and improves dimension reduction of scRNAseq datasets."
- [intro] corral interfaces with Bioconductor classes: "We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes"
