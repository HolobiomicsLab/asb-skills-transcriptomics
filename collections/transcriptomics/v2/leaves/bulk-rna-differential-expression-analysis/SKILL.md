---
name: bulk-rna-differential-expression-analysis
description: Use when you need to rebuild or validate the gene feature panel behind
  the sc_xgboost malignancy classifier, e.g. you have TCGA bulk counts with malignant/normal
  labels plus an integrated scRNA-seq matrix and must reproduce GbulkDE ∪ GscHV.
when_to_use_negative:
- You already have the scCancer2 reference matrix or genes-scRNA-tcga-sorted.txt shipped
  with the package (system.file('txt', package='scCancer')) — no reconstruction is
  needed
- Your goal is cell-subtype annotation with predCellType or runCellSubtypeClassify,
  which uses marker/submodel paths rather than this TCGA-derived panel
- You lack malignant/normal bulk sample labels, since the entire panel derives from
  edgeR contrasts between TCGA malignant and normal samples
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: edgeR
  role: Runs the per-cancer-type differential expression tests between malignant and
    normal TCGA bulk samples, producing the top 5000 gene lists
- name: scCancer2
  role: Host R package whose sc_xgboost malignancy module loads genes-scRNA-tcga-sorted.txt
    and trains/predicts on the 2,756-gene panel
  repo: https://doi.org/10.5281/zenodo.10477296
- name: Seurat
  role: Provides the integrated single-cell matrix from which the 2000 high-variance
    genes (GscHV) are computed and to which the panel subset is applied
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:3df3f0e9a08f46ae2c29c183963b1dcce1d7e02c2556cd9224ff04f2dfe74bab
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - edgeR
  - scCancer
  - scCancer2
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# bulk-rna-differential-expression-analysis

## Summary

Reconstructs scCancer2's bulk+single-cell feature selection: edgeR differential expression between malignant and normal TCGA bulk samples across 14 cancer types is combined with high-variance scRNA-seq genes to produce the 2,756-gene panel used to train the malignant cell classifier on a 466,468-cell reference matrix.

## When to use

Use when you need to rebuild or validate the gene feature panel behind the sc_xgboost malignancy classifier, e.g. you have TCGA bulk counts with malignant/normal labels plus an integrated scRNA-seq matrix and must reproduce GbulkDE ∪ GscHV.

## When NOT to use

- You already have the scCancer2 reference matrix or genes-scRNA-tcga-sorted.txt shipped with the package (system.file('txt', package='scCancer')) — no reconstruction is needed
- Your goal is cell-subtype annotation with predCellType or runCellSubtypeClassify, which uses marker/submodel paths rather than this TCGA-derived panel
- You lack malignant/normal bulk sample labels, since the entire panel derives from edgeR contrasts between TCGA malignant and normal samples

## Inputs

- TCGA bulk RNA-seq count matrices with malignant/normal sample labels for 14 cancer types
- Integrated single-cell RNA-seq count matrix spanning 466,468 cells (scCancer2 malignancy reference)
- Gene symbol annotations for the integrated single-cell matrix

## Outputs

- GbulkDE: the 1,997-gene recurrent bulk DE set (1,069 genes after intersecting with the single-cell matrix)
- GscHV: the 2000 highest-variance single-cell genes
- genes-scRNA-tcga-sorted.txt: the sorted 2,756-gene list consumed by the xgboost malignancy classifier
- Reference matrix: 466,468 cells × 2,756 genes

## How to apply

For each of the 14 cancer types, load the TCGA bulk RNA-seq count matrix with malignant/normal sample labels and run edgeR differential expression, retaining the top 5000 genes per cancer-type comparison. Tally recurrence of each gene across cancer types and keep only genes appearing in more than 8 cancer types to form GbulkDE (1,997 genes), then intersect GbulkDE with gene symbols present in the integrated 466,468-cell scRNA-seq matrix (yielding 1,069 genes). Independently compute per-gene variance across the integrated scRNA-seq matrix and select the top 2000 high-variance genes as GscHV; take the union GbulkDE ∪ GscHV to obtain the final 2,756-gene panel. Subset the integrated scRNA-seq matrix to this panel to build the 466,468-cell × 2,756-gene reference matrix, and write the sorted gene list to genes-scRNA-tcga-sorted.txt — the file the downstream sc_xgboost classifier loads via read.table and zero-fills for missing features.

## Related tools

- **edgeR** (Runs the per-cancer-type differential expression tests between malignant and normal TCGA bulk samples, producing the top 5000 gene lists)
- **scCancer2** (Host R package whose sc_xgboost malignancy module loads genes-scRNA-tcga-sorted.txt and trains/predicts on the 2,756-gene panel) — https://doi.org/10.5281/zenodo.10477296
- **Seurat** (Provides the integrated single-cell matrix from which the 2000 high-variance genes (GscHV) are computed and to which the panel subset is applied)

## Examples

```
genes.path <- paste0(system.file("txt", package = "scCancer"), "/genes-scRNA-tcga-sorted.txt")
```

## Evaluation signals

- Final gene panel contains exactly 2,756 genes
- Reference matrix dimensions are 466,468 cells × 2,756 genes, matching the matrix used by the sc_xgboost malignancy classifier
- Per cancer type, the edgeR DE list retains exactly the top 5000 genes, and GbulkDE keeps only genes recurring in more than 8 of the 14 cancer types
- genes-scRNA-tcga-sorted.txt loads cleanly via read.table and the classifier zero-fills any features missing in a new query matrix without error
- GscHV corresponds to the top 2000 genes by variance computed on the integrated single-cell matrix

## Limitations

- The panel is derived from pan-cancer TCGA bulk data; per-cancer-type composition of the >8-of-14 recurrence cutoff is not itself a per-tumor guarantee, so rare-cancer-specific markers may be excluded
- The classifier is designed for the no-internal-reference scenario; when a confidential internal malignant reference is available, alternative malignancy estimation (e.g. inferCNV-style) may be preferable
- Missing features in a query dataset are zero-filled at prediction time, so scRNA-seq data whose gene symbols do not overlap the panel degrade silently

## Evidence

- [abstract] integrating large-scale pan-cancer TCGA bulk gene expression datasets and scRNA-seq datasets (10 cancer types, 175 samples, 663 857 cells): "integrating large-scale pan-cancer TCGA bulk gene expression datasets and scRNA-seq datasets (10 cancer types, 175 samples, 663 857 cells)"
- [methods] genes.path: "genes.path <- paste0(system.file("txt", package = "scCancer"), "/genes-scRNA-tcga-sorted.txt")"
- [methods] variable feature selection: "object <- FindVariableFeatures(object, selection.method = "vst", nfeatures = min(5000, length(rownames(object))), verbose = FALSE)"
- [readme] malignant cell reference: [evidence span withheld — anchor status: external_doc]
- [abstract] classifier robustness: "This classifier shows robust performances when no internal confidential reference cells are available."
