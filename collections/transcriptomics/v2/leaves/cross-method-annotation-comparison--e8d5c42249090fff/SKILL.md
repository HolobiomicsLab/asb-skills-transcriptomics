---
name: cross-method-annotation-comparison
description: Use when several prior-knowledge cell-type annotation methods must be
  compared under identical marker-database perturbations, e.g. quantifying how often
  each method falsely assigns a removed (unknown) cell type.
when_to_use_negative:
- scCATCH must be excluded from this comparison because its implementation does not
  allow custom marker sets
- The task is not marker-perturbation sensitivity but general annotation accuracy
  on fully labeled reference data — supervised methods (CaSTLe, ACTINN) trained on
  annotated datasets are out of this skill's unsupervised scope
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: ScType
  role: Primary annotation method; run via gene_sets_prepare() and sctype_score()
    with positive/negative markers and the ncells/4 'Unknown' rule
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: SCINA
  role: Comparison annotation method re-run with identical perturbed marker sets
- name: scSorter
  role: Comparison annotation method re-run with identical perturbed marker sets
- name: Seurat
  role: Seurat v4.0 used for data processing and normalization to produce the scaled
    expression matrix input
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:3389748234527ec68f2fb8de035f216a98473f9bd78cbb188336b24a4b7bb2ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - R
  - SCINA
  - ScType
  - Seurat
  - scSorter
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cross-method-annotation-comparison

## Summary

Benchmark multiple marker-based scRNA-seq cell-type annotation methods (ScType, SCINA, scSorter) side-by-side by re-running each with identical perturbed marker databases on the same datasets and comparing their ability to leave a removed cell type undetected.

## When to use

Use when several prior-knowledge cell-type annotation methods must be compared under identical marker-database perturbations, e.g. quantifying how often each method falsely assigns a removed (unknown) cell type.

## When NOT to use

- scCATCH must be excluded from this comparison because its implementation does not allow custom marker sets
- The task is not marker-perturbation sensitivity but general annotation accuracy on fully labeled reference data — supervised methods (CaSTLe, ACTINN) trained on annotated datasets are out of this skill's unsupervised scope

## Inputs

- 45 simulated scRNA-seq count matrices (one per dropout perturbation)
- Ground-truth cell-type label of the perturbed type per dataset
- Marker database in ScTypeDB-format XLSX (tissueType, cellName, geneSymbolmore1, geneSymbolmore2)
- Scaled/normalized expression matrix (Seurat object, scaled=TRUE)

## Outputs

- Per-dataset detection outcome table (method x dataset: undetected/unknown vs. assigned)
- Aggregated unknown-detection counts per method (e.g. ScType 43/45, SCINA 41/45, scSorter 22/45)
- Comparison rates with percentages (95.5%, 91.1%, 48.8%)

## How to apply

Load the simulated scRNA-seq count matrices (here 45 dropout-simulated datasets, one per perturbation) together with the ground-truth label of the perturbed cell type. For each dataset, edit the marker database in ScTypeDB XLSX format (columns tissueType, cellName, geneSymbolmore1 positive markers, geneSymbolmore2 negative markers) so that exactly one cell type's markers are removed at a time via gene_sets_prepare(). Run ScType's sctype_score() on the scaled expression matrix with positive (gs) and negative (gs2) marker sets, applying the low-confidence rule that assigns 'Unknown' when the top score falls below ncells/4, then run SCINA and scSorter with the identical perturbed marker sets. Record per method and dataset whether the perturbed cell type is left undetected/unknown, aggregate over all 45 datasets, and compare against expected counts (ScType 43/45, SCINA 41/45, scSorter 22/45).

## Related tools

- **ScType** (Primary annotation method; run via gene_sets_prepare() and sctype_score() with positive/negative markers and the ncells/4 'Unknown' rule) — https://github.com/IanevskiAleksandr/sc-type
- **SCINA** (Comparison annotation method re-run with identical perturbed marker sets)
- **scSorter** (Comparison annotation method re-run with identical perturbed marker sets)
- **Seurat** (Seurat v4.0 used for data processing and normalization to produce the scaled expression matrix input)

## Examples

```
gs_list <- gene_sets_prepare("ScTypeDB_short.xlsx", "Immune system"); es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)
```

## Evaluation signals

- Reproduced unknown-detection counts match the reported values: ScType 43/45 (95.5%), SCINA 41/45 (91.1%), scSorter 22/45 (48.8%)
- Each of the 45 datasets yields exactly one outcome per method (undetected vs. assigned) — no missing or extra rows
- ScType results are consistent with the low-confidence rule: any assignment with top score below ncells/4 is recorded as 'Unknown'
- All three methods were run with the same per-dataset perturbed marker sets derived from the same ScTypeDB XLSX, ensuring a controlled comparison

## Limitations

- Drop-out events in scRNA-seq data are not corrected in the current version, which the article notes affects identification of true negative markers
- Z-score transformation used by ScType may be suboptimal for single-cell transcriptomic data
- The number of negative markers is still relatively low compared to positive markers, limiting annotation power
- scCATCH cannot be included in the comparison since its implementation does not allow custom markers

## Evidence

- [other] Low-confidence Unknown rule: [evidence span withheld — anchor status: not_found]
- [other] Marker database format: [evidence span withheld — anchor status: not_found]
- [discussion] Seurat processing: "We utilized Seurat v4.042 for data processing and normalization."
- [discussion] ScType outperformed marker-based methods: "Our unsupervised approach outperformed the prior knowledge-based methods scSorter30, SCINA33, and scCATCH34"
- [intro] Positive and negative markers incorporated: "the expression of negative marker genes, which provide evidence against a cell being of a particular type, should also be incorporated into the cell-type identification process"
- [readme] gene_sets_prepare usage: "gs_list <- gene_sets_prepare("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/ScTypeDB_short.xlsx", "Immune system")"
