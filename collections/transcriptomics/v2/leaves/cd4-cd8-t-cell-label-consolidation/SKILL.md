---
name: cd4-cd8-t-cell-label-consolidation
description: Use when cell-type predictions from scCancer2 predCellType (or a reference
  metadata scheme) emit CD4/CD8-split T-cell labels but the evaluation targets a coarse
  five-major-cell-type taxonomy across datasets and sequencing technologies.
when_to_use_negative:
- When the evaluation taxonomy itself distinguishes CD4 vs CD8 T-cell subtypes (fine-grained
  annotation), since merging would destroy the label of interest
- When input labels are already at the five-major-cell-type level and contain no CD4/CD8-split
  T-cell annotations
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scCancer2
  role: Source of predCellType labels and the annotation framework whose benchmark
    labels are consolidated
  repo: https://doi.org/10.5281/zenodo.10477296
- name: scCancer
  role: R package providing predCellType used to produce the rough labels being merged
  repo: https://github.com/czythu/scCancer
- name: Seurat
  role: Holds benchmark datasets as Seurat objects whose 'RNA' @data assay feeds predCellType
- name: dplyr
  role: Loaded alongside scCancer for label manipulation and tabulation of consolidated
    labels
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:7ec351cf5e06bc178ade7ac038b7f9108131b8c8817da542e24a5b64fe3d6708
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHETAH
  - OCLR
  - R
  - Scibet
  - Seurat
  - SingleCellNet
  - dplyr
  - scCancer
  - scCancer2
  - scPred
  - scmap
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# CD4/CD8 T-cell label consolidation

## Summary

A label-harmonization step within the scCancer2 benchmark workflow that merges fine-grained T-cell labels ('T.cells.CD4' and 'T.cells.CD8') into a unified 'T.cells' category so predicted annotations can be compared against a five-type level-1 reference (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial).

## When to use

Use when cell-type predictions from scCancer2 predCellType (or a reference metadata scheme) emit CD4/CD8-split T-cell labels but the evaluation targets a coarse five-major-cell-type taxonomy across datasets and sequencing technologies.

## When NOT to use

- When the evaluation taxonomy itself distinguishes CD4 vs CD8 T-cell subtypes (fine-grained annotation), since merging would destroy the label of interest
- When input labels are already at the five-major-cell-type level and contain no CD4/CD8-split T-cell annotations

## Inputs

- Seurat object with normalized expression (assay slot 'RNA' @data) loaded from per-study .RDS benchmark files
- Predicted cell type labels from scCancer2 predCellType (t.results$type.pred)
- Level-1 reference cell type labels from dataset metadata (five types: T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial)

## Outputs

- Consolidated predicted label vector with 'T.cells.CD4'/'T.cells.CD8' merged into 'T.cells'
- Intersection of predicted and reference types restricted to the five-type list, ready for accuracy computation in accuracy_table.csv

## How to apply

After running scCancer2's predCellType on the normalized expression matrix (Seurat object assay 'RNA' @data), collect the predicted type labels. Use string substitution (e.g. gsub in R) to collapse the 'T.cells.CD4' and 'T.cells.CD8' labels into a single 'T.cells' label, then intersect the consolidated predicted types with the five-type default list (T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial) so that predictions and reference metadata share the same level-1 vocabulary. This consolidation is a prerequisite for computing per-method, per-dataset prediction accuracy against level-1 reference labels in the benchmark comparison of scCancer2 against scPred, OCLR, multinomial, Scibet, CHETAH, scmap, and SingleCellNet. Rationale: heterogeneous public atlases use different label granularities, so coarse-graining T-cell subtypes guarantees a one-to-one match with the reference taxonomy before accuracy is tabulated.

## Related tools

- **scCancer2** (Source of predCellType labels and the annotation framework whose benchmark labels are consolidated) — https://doi.org/10.5281/zenodo.10477296
- **scCancer** (R package providing predCellType used to produce the rough labels being merged) — https://github.com/czythu/scCancer
- **Seurat** (Holds benchmark datasets as Seurat objects whose 'RNA' @data assay feeds predCellType)
- **dplyr** (Loaded alongside scCancer for label manipulation and tabulation of consolidated labels)

## Examples

```
t.results <- predCellType(expr@assays[["RNA"]]@data); expr$Cell.Type <- gsub("T\\.cells\\.(CD4|CD8)", "T.cells", t.results$type.pred); expr$Cell.Type <- intersect(expr$Cell.Type, c("T.cells","Myeloid.cells","B.cells","Fibroblast","Endothelial"))
```

## Evaluation signals

- After merging, no predicted label contains the substrings 'CD4' or 'CD8'
- Consolidated label set intersected with the default list yields at most five unique types: T.cells, Myeloid.cells, B.cells, Fibroblast, Endothelial
- Every predicted label after consolidation has a non-empty match in the level-1 reference metadata, enabling one accuracy entry per method per benchmark dataset
- Resulting accuracy_table.csv contains one accuracy value per method (scCancer2 and the seven baselines) per dataset/sequencing technology, matching the Fig. 2B structure

## Limitations

- No software version numbers are stated for R, scCancer, Seurat, or xgboost in the methods, so exact reproducibility of label vocabularies may vary across package versions
- No repository URL or accession IDs are provided for the HCC benchmark datasets, so the exact reference label schemes must be confirmed from metadata
- Merging CD4/CD8 into 'T.cells' discards subtype resolution and is only appropriate for the five-major-cell-type benchmark framing

## Evidence

- [methods] Rough label annotation with predCellType: "t.results <- predCellType(expr@assays[["RNA"]]@data)
expr$Cell.Type <- t.results$type.pred"
- [other] Benchmark finding from source card workflow: [evidence span withheld — anchor status: not_found]
- [results] Benchmark comparison result: [evidence span withheld — anchor status: not_found]
