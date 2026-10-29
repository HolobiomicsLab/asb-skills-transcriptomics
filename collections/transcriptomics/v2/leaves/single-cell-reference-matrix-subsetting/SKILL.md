---
name: single-cell-reference-matrix-subsetting
description: Use when building or reproducing a malignancy-classifier gene panel by
  intersecting pan-cancer bulk RNA-seq DE genes (e.g. TCGA edgeR top-5000 lists) with
  high-variance genes from an integrated scRNA-seq matrix, and subsetting that matrix
  into a classifier-ready reference.
when_to_use_negative:
- The input is already a trained feature panel or the downstream classifier's genes-scRNA-tcga-sorted.txt
  already exists with the expected 2,756 genes — no reconstruction is needed.
- The goal is generic variable-feature selection within a single scRNA-seq dataset
  (e.g. Seurat FindVariableFeatures alone) rather than a bulk+single-cell integrated
  malignancy reference.
- A confident internal scRNA-seq reference with labeled malignant and non-malignant
  cells is available, making the TCGA-bulk-derived reference unnecessary.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: edgeR
  role: Runs per-cancer-type differential expression between malignant and normal
    TCGA bulk samples to produce the top-5000 DE gene lists
- name: Seurat
  role: Hosts the integrated scRNA-seq object from which per-gene variance is computed
    and the 466,468-cell matrix is subset to the 2,756-gene panel
- name: scCancer2
  role: Provides genes-scRNA-tcga-sorted.txt (loaded via system.file('txt', package='scCancer'))
    and the xgboost classifier that consumes the subsetted reference matrix
  repo: http://lifeome.net/software/sccancer2/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:c65b5bffccb558ebd5fa02fc5d41497af034ac834f936f236b26e5f72f0ee1bf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - edgeR
  - scCancer
  - scCancer2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# single-cell-reference-matrix-subsetting

## Summary

Reconstructs scCancer2's bulk+single-cell feature-selection pipeline that builds the gene panel (GbulkDE ∪ GscHV, 2,756 genes) for the malignant-cell XGBoost reference matrix of 466,468 cells. It bridges pan-cancer TCGA bulk differential expression with single-cell high-variance features so the classifier works when no internal confident reference cells exist.

## When to use

Use when building or reproducing a malignancy-classifier gene panel by intersecting pan-cancer bulk RNA-seq DE genes (e.g. TCGA edgeR top-5000 lists) with high-variance genes from an integrated scRNA-seq matrix, and subsetting that matrix into a classifier-ready reference.

## When NOT to use

- The input is already a trained feature panel or the downstream classifier's genes-scRNA-tcga-sorted.txt already exists with the expected 2,756 genes — no reconstruction is needed.
- The goal is generic variable-feature selection within a single scRNA-seq dataset (e.g. Seurat FindVariableFeatures alone) rather than a bulk+single-cell integrated malignancy reference.
- A confident internal scRNA-seq reference with labeled malignant and non-malignant cells is available, making the TCGA-bulk-derived reference unnecessary.

## Inputs

- TCGA bulk RNA-seq count matrix with malignant/normal sample labels per cancer type (14 cancer types)
- Integrated scRNA-seq expression matrix (466,468 cells) with gene-symbol row names

## Outputs

- Final reference matrix of 466,468 cells x 2,756 genes (GbulkDE ∪ GscHV panel)
- Sorted gene-list file genes-scRNA-tcga-sorted.txt consumed by the sc_xgboost malignancy classifier

## How to apply

Load TCGA bulk RNA-seq counts with malignant/normal labels for each cancer type (14 in the reference build) and run edgeR differential expression per cancer type, keeping the top 5000 DE genes per comparison. Tally gene recurrence across cancer types and retain only genes appearing in more than 8 cancer types to form GbulkDE; intersect GbulkDE with gene symbols present in the integrated scRNA-seq matrix (1,069 of 1,997 survive). Separately compute per-gene variance across the integrated scRNA-seq matrix and take the top 2000 genes as GscHV (mirroring Seurat FindVariableFeatures with selection.method='vst', nfeatures=5000-style logic), then take the union GbulkDE ∪ GscHV to obtain the final 2,756-gene panel. Subset the integrated 466,468-cell scRNA-seq matrix to this panel to build the reference matrix, and write the sorted gene list to genes-scRNA-tcga-sorted.txt — the file the downstream xgboost malignancy classifier loads via read.table, zero-filling features missing from a query dataset.

## Related tools

- **edgeR** (Runs per-cancer-type differential expression between malignant and normal TCGA bulk samples to produce the top-5000 DE gene lists)
- **Seurat** (Hosts the integrated scRNA-seq object from which per-gene variance is computed and the 466,468-cell matrix is subset to the 2,756-gene panel)
- **scCancer2** (Provides genes-scRNA-tcga-sorted.txt (loaded via system.file('txt', package='scCancer')) and the xgboost classifier that consumes the subsetted reference matrix) — http://lifeome.net/software/sccancer2/

## Examples

```
genes.path <- paste0(system.file("txt", package = "scCancer"), "/genes-scRNA-tcga-sorted.txt")
```

## Evaluation signals

- The final gene panel contains exactly 2,756 genes.
- The subsetted reference matrix spans exactly 466,468 cells and 2,756 genes, matching the dimensions used by the sc_xgboost malignancy classifier.
- GbulkDE contains only genes appearing in more than 8 of the 14 TCGA cancer-type DE comparisons (1,997 genes before single-cell intersection, 1,069 after).
- The written genes-scRNA-tcga-sorted.txt is loadable via read.table by the downstream classifier and missing query features are zero-filled.

## Limitations

- The bulk-derived component is cancer-type dependent: only genes recurrently DE across >8 of the 14 TCGA cancer types are retained, so rare or subtype-specific malignant markers may be excluded.
- Only 1,069 of the 1,997 recurrent bulk DE genes are present in the single-cell matrix; the remainder are lost at the gene-symbol intersection step.
- The downstream classifier zero-fills missing features from query datasets, so queries with poor gene-symbol overlap will silently degrade in information content.

## Evidence

- [methods] The feature-selection mechanism computes the top 5000 edgeR DE genes between malignant and normal TCGA bulk samples across 14 cancer types: [evidence span withheld — anchor status: not_found]
- [methods] unions GbulkDE with GscHV: [evidence span withheld — anchor status: not_found]
- [readme] Malignant cell identification: [evidence span withheld — anchor status: external_doc]
- [methods] genes.path definition: "genes.path <- paste0(system.file("txt", package = "scCancer"), "/genes-scRNA-tcga-sorted.txt")"
- [methods] Variable feature selection: "object <- FindVariableFeatures(object, selection.method = "vst", nfeatures = min(5000, length(rownames(object))), verbose = FALSE)"
