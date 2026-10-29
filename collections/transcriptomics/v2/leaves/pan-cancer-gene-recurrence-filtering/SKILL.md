---
name: pan-cancer-gene-recurrence-filtering
description: Use when building a cross-cancer malignant-cell classifier reference
  and you need a gene panel selected from recurrent bulk-RNA-seq DE signals plus high-variance
  single-cell genes.
when_to_use_negative:
- The goal is cancer-type-specific differential expression rather than a pan-cancer
  reference gene panel shared across tumor types
- Input already contains a fixed feature/gene panel or the downstream classifier expects
  its own feature schema
- No paired malignant/normal bulk samples exist for the cancer types under study,
  so per-type edgeR comparisons cannot be run
tools:
- name: edgeR
  role: Performs per-cancer-type differential expression between malignant and normal
    TCGA bulk samples to produce the top 5000 DE genes per comparison
- name: scCancer2
  role: Hosts the feature-selection routine and ships the resulting gene list (genes-scRNA-tcga-sorted.txt)
    loaded by the malignancy classifier
  repo: czythu/scCancer
- name: Seurat
  role: Used in the scCancer2 pipeline for single-cell normalization and variable-feature
    workflows from which high-variance genes (GscHV) derive
- name: R
  role: Implementation language for the edgeR-based selection and reference-matrix
    construction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:93607c749065efe33b47a8fd040dd4cde686e8e2eb67f68584f4dce284eff5d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/czythu/scCancer
  tools:
  - R
  - Seurat
  - edgeR
  - scCancer
  - scCancer2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# pan-cancer-gene-recurrence-filtering

## Summary

Derives a compact pan-cancer gene panel by tallying recurrent top edgeR differential-expression genes between malignant and normal TCGA bulk samples across cancer types, then unifying them with high-variance scRNA-seq genes. The panel (GbulkDE ∪ GscHV, 2,756 genes) backs scCancer2's 466,468-cell malignancy reference matrix.

## When to use

Use when building a cross-cancer malignant-cell classifier reference and you need a gene panel selected from recurrent bulk-RNA-seq DE signals plus high-variance single-cell genes.

## When NOT to use

- The goal is cancer-type-specific differential expression rather than a pan-cancer reference gene panel shared across tumor types
- Input already contains a fixed feature/gene panel or the downstream classifier expects its own feature schema
- No paired malignant/normal bulk samples exist for the cancer types under study, so per-type edgeR comparisons cannot be run

## Inputs

- TCGA bulk RNA-seq count data with malignant/normal sample labels for each of 14 cancer types
- Integrated scRNA-seq count/expression matrix (466,468 cells) from 15 reference datasets
- edgeR R package for per-cancer-type differential expression

## Outputs

- GbulkDE gene list (1997 recurrent bulk DE genes, 1069 present in the single-cell matrix)
- GscHV gene list (top 2000 highest-variance scRNA-seq genes)
- Final 2,756-gene panel (GbulkDE ∪ GscHV) written to genes-scRNA-tcga-sorted.txt
- Reference matrix of 466,468 cells x 2,756 genes for the malignancy classifier

## How to apply

Run edgeR differential expression between malignant and normal TCGA bulk samples independently within each of 14 cancer types, retaining the top 5000 DE genes per comparison. Tally gene recurrence across cancer types and keep only genes appearing in more than 8 cancer types to form GbulkDE (1997 genes, of which 1069 are present in the single-cell matrix). Intersect GbulkDE with gene symbols in the integrated scRNA-seq matrix, compute per-gene variance across that matrix, take the top 2000 high-variance genes as GscHV, and form the union GbulkDE ∪ GscHV. Subset the integrated 466,468-cell scRNA-seq matrix to the resulting 2,756-gene panel to build the reference matrix used by the sc_xgboost malignancy classifier, writing the sorted gene list to genes-scRNA-tcga-sorted.txt. The recurrence threshold (>8/14 types) is the rationale for robustness: genes must be consistently malignant-associated across cancers rather than tumor-type-specific.

## Related tools

- **edgeR** (Performs per-cancer-type differential expression between malignant and normal TCGA bulk samples to produce the top 5000 DE genes per comparison)
- **scCancer2** (Hosts the feature-selection routine and ships the resulting gene list (genes-scRNA-tcga-sorted.txt) loaded by the malignancy classifier) — czythu/scCancer
- **Seurat** (Used in the scCancer2 pipeline for single-cell normalization and variable-feature workflows from which high-variance genes (GscHV) derive)
- **R** (Implementation language for the edgeR-based selection and reference-matrix construction)

## Examples

```
genes.path <- paste0(system.file("txt", package = "scCancer"), "/genes-scRNA-tcga-sorted.txt")
```

## Evaluation signals

- The final gene panel contains exactly 2,756 genes
- The reference matrix spans exactly 466,468 cells, matching the sc_xgboost malignancy classifier reference
- Every retained GbulkDE gene appears in more than 8 of the 14 cancer-type edgeR comparisons
- 1069 of the 1997 recurrent bulk DE genes are present in the integrated single-cell matrix before unioning with GscHV
- The sorted gene list file genes-scRNA-tcga-sorted.txt is loadable via read.table and supports zero-filling of missing features at prediction time

## Limitations

- The recurrence cutoff (>8 of 14 cancer types) excludes tumor-type-specific malignancy markers, so the panel may miss cancer-exclusive signals
- Only 1069 of the 1997 recurrent bulk DE genes overlap the single-cell matrix; missing genes are handled by zero-filling in the classifier, which dilutes bulk-derived signal
- No software version numbers are stated for R, scCancer, Seurat, or xgboost, complicating exact reproduction
- No repository URL or accession IDs are provided for the HCC scRNA-seq datasets used in the reference

## Evidence

- [results] top 5000 edgeR DE genes between malignant and normal TCGA bulk samples across 14 cancer types: [evidence span withheld — anchor status: not_found]
- [results] genes appearing >8 times retained as GbulkDE: [evidence span withheld — anchor status: not_found]
- [results] union of GbulkDE and GscHV yields the final reference matrix: [evidence span withheld — anchor status: not_found]
- [methods] genes-scRNA-tcga-sorted.txt loaded by the classifier: "genes.path <- paste0(system.file("txt", package = "scCancer"), "/genes-scRNA-tcga-sorted.txt")"
- [readme] scCancer2 README on combined scRNA-seq and bulk RNA-seq reference: [evidence span withheld — anchor status: external_doc]
