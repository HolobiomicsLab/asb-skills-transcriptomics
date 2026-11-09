---
name: cross-platform-gene-symbol-intersection
description: Use when a downstream model (e.g. an xgboost malignancy classifier) requires
  a reference gene panel that must be reconciled between TCGA bulk RNA-seq feature
  space and single-cell count matrix gene symbols.
when_to_use_negative:
- Bulk and single-cell data already share an identical gene identifier namespace with
  no reference-building step needed — a simple subset suffices
- Input is already a finalized feature table of the exact dimensionality the downstream
  model expects (no reconciliation required)
- The single-cell matrix uses non-standard or one-to-many gene identifiers (e.g. Ensembl
  IDs without symbol mapping) that cannot be matched to bulk DE gene symbols
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scCancer
  role: R package hosting the sorted gene list (genes-scRNA-tcga-sorted.txt) that
    the malignancy classifier loads; the reference matrix is built against its single-cell
    annotation workflow
- name: edgeR
  role: Differential expression between malignant and normal TCGA bulk samples per
    cancer type to generate the top 5000 candidate genes per comparison
- name: Seurat
  role: Manages the integrated scRNA-seq matrix from which gene symbols are intersected
    and per-gene variance (vst, 5000 features) is computed for GscHV
- name: xgboost
  role: Downstream consumer of the reconciled gene panel; the pretrained malignancy
    classifier zero-fills features missing at prediction time via the sorted gene
    list
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:3bb10e5c09f7912927447d836a0d2a0075ed6178db6475a78adaaccdf61db94a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - edgeR
  - scCancer
  - scCancer2
  - xgboost
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# cross-platform-gene-symbol-intersection

## Summary

Reconciles gene features between bulk RNA-seq-derived panels and a single-cell expression matrix by intersecting/unioning gene symbol lists, as scCancer2 does to build its malignant-cell reference from TCGA bulk DE genes plus high-variance scRNA-seq genes.

## When to use

Use when a downstream model (e.g. an xgboost malignancy classifier) requires a reference gene panel that must be reconciled between TCGA bulk RNA-seq feature space and single-cell count matrix gene symbols.

## When NOT to use

- Bulk and single-cell data already share an identical gene identifier namespace with no reference-building step needed — a simple subset suffices
- Input is already a finalized feature table of the exact dimensionality the downstream model expects (no reconciliation required)
- The single-cell matrix uses non-standard or one-to-many gene identifiers (e.g. Ensembl IDs without symbol mapping) that cannot be matched to bulk DE gene symbols

## Inputs

- TCGA bulk RNA-seq count data with malignant/normal sample labels, stratified by cancer type (14 cancer types)
- Integrated single-cell RNA-seq count matrix (e.g. 466,468 cells x gene symbols, loaded as a Seurat object)
- edgeR differential expression results ranked per cancer type

## Outputs

- GbulkDE gene list (1997 recurrent bulk DE genes, of which 1069 map to the single-cell matrix)
- GscHV gene list (top 2000 high-variance scRNA-seq genes)
- Final 2,756-gene reference panel (GbulkDE ∪ GscHV)
- Subset reference matrix of 466,468 cells x 2,756 genes
- Sorted gene-symbol file genes-scRNA-tcga-sorted.txt consumed by the sc_xgboost classifier

## How to apply

Run edgeR differential expression between malignant and normal TCGA bulk samples within each of the 14 cancer types, retain the top 5000 DE genes per comparison, and tally each gene's recurrence across cancer types, keeping only genes found in more than 8 cancer types to form GbulkDE. Intersect GbulkDE with the gene symbols present in the integrated 466,468-cell scRNA-seq matrix so only features expressible in the single-cell space survive. Separately compute per-gene variance across the integrated scRNA-seq matrix and select the top 2000 high-variance genes as GscHV (FindVariableFeatures with vst). Take the union GbulkDE ∪ GscHV to obtain the final 2,756-gene panel, subset the integrated scRNA-seq matrix to it, and write the sorted gene list (genes-scRNA-tcga-sorted.txt) that the downstream classifier loads via read.table and zero-fills for missing features. The rationale is that bulk-derived discriminative features only help a single-cell classifier if their symbols exist in the single-cell count matrix, and high-variance single-cell genes compensate for bulk features lost in cross-platform mapping.

## Related tools

- **scCancer** (R package hosting the sorted gene list (genes-scRNA-tcga-sorted.txt) that the malignancy classifier loads; the reference matrix is built against its single-cell annotation workflow)
- **edgeR** (Differential expression between malignant and normal TCGA bulk samples per cancer type to generate the top 5000 candidate genes per comparison)
- **Seurat** (Manages the integrated scRNA-seq matrix from which gene symbols are intersected and per-gene variance (vst, 5000 features) is computed for GscHV)
- **xgboost** (Downstream consumer of the reconciled gene panel; the pretrained malignancy classifier zero-fills features missing at prediction time via the sorted gene list)

## Examples

```
genes.path <- paste0(system.file("txt", package = "scCancer"), "/genes-scRNA-tcga-sorted.txt")
```

## Evaluation signals

- Final gene panel contains exactly 2,756 genes (GbulkDE ∪ GscHV)
- Reference matrix dimensions equal 466,468 cells x 2,756 genes, matching the sc_xgboost malignancy classifier reference
- Every retained GbulkDE gene symbol is present in the integrated single-cell matrix (symbol-level intersection, not just bulk recurrence)
- genes-scRNA-tcga-sorted.txt loads via read.table and zero-fills missing features without errors at classifier invocation
- Recurrent bulk DE set is restricted to genes appearing in more than 8 of the 14 TCGA cancer types

## Limitations

- Cross-platform symbol loss is substantial: only 1069 of the 1997 recurrent bulk DE genes are present in the integrated single-cell matrix, so most bulk features cannot be transferred
- No software version numbers are stated for R, scCancer, Seurat, or edgeR, complicating exact reproduction
- Features missing at classification time are zero-filled rather than imputed, so symbol mismatches silently degrade the classifier input

## Evidence

- [readme] Malignant cell identification: We constructed a reference dataset combining scRNA-seq and bulk RNA-seq data across multiple cancer types: [evidence span withheld — anchor status: external_doc]
- [methods] genes.path <- paste0(system.file("txt", package = "scCancer"), "/genes-scRNA-tcga-sorted.txt"): "genes.path <- paste0(system.file("txt", package = "scCancer"), "/genes-scRNA-tcga-sorted.txt")"
- [methods] object <- FindVariableFeatures(object, selection.method = "vst", nfeatures = min(5000, length(rownames(object))), verbose = FALSE): "object <- FindVariableFeatures(object, selection.method = "vst", nfeatures = min(5000, length(rownames(object))), verbose = FALSE)"
- [abstract] integrating large-scale pan-cancer TCGA bulk gene expression datasets and scRNA-seq datasets (10 cancer types, 175 samples, 663 857 cells): "integrating large-scale pan-cancer TCGA bulk gene expression datasets and scRNA-seq datasets (10 cancer types, 175 samples, 663 857 cells)"
