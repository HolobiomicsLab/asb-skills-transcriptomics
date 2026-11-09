---
name: simulated-dropout-benchmark-evaluation
description: Use when evaluating how marker-based annotators behave when a cell type's
  marker signatures are deleted from a ScTypeDB-format marker database, and you must
  score which methods correctly report Unknown versus misassigning the perturbed type.
when_to_use_negative:
- Comparing methods that cannot accept custom/perturbed marker sets — e.g. scCATCH,
  which the paper excluded 'since its implementation does not allow for the use custom
  markers'
- Standard annotation of real (non-simulated) scRNA-seq data with the intact, unperturbed
  ScType database — this skill is specifically the leave-one-cell-type-out unknown-detection
  benchmark
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ScType
  role: 'Primary annotator being benchmarked: run sctype_score() with positive (gs)
    and negative (gs2) marker sets after perturbing its marker database'
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: SCINA
  role: Comparator method rerun with the identical perturbed marker sets on the 45
    simulated datasets
- name: scSorter
  role: Comparator method rerun with the identical perturbed marker sets on the 45
    simulated datasets
- name: Seurat
  role: Data processing and normalization producing the scaled expression matrix fed
    to sctype_score()
- name: R
  role: Implementation language; gene_sets_prepare.R and sctype_score_.R are sourced
    as R functions
  repo: https://github.com/IanevskiAleksandr/sc-type
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:3e3b66382c2d2b57d558ab8421da72cfd9a3f7bf122fc829bd1b32b6ace05c21
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# simulated-dropout-benchmark-evaluation

## Summary

Benchmarks marker-based cell-type annotators (ScType, SCINA, scSorter) for their ability to leave a cell type undetected/unknown when its marker signatures are removed, using 45 simulated dropout scRNA-seq datasets in a leave-one-cell-type-out design.

## When to use

Use when evaluating how marker-based annotators behave when a cell type's marker signatures are deleted from a ScTypeDB-format marker database, and you must score which methods correctly report Unknown versus misassigning the perturbed type.

## When NOT to use

- Comparing methods that cannot accept custom/perturbed marker sets — e.g. scCATCH, which the paper excluded 'since its implementation does not allow for the use custom markers'
- Standard annotation of real (non-simulated) scRNA-seq data with the intact, unperturbed ScType database — this skill is specifically the leave-one-cell-type-out unknown-detection benchmark

## Inputs

- 45 simulated scRNA-seq count matrices from the dropout simulation (one per perturbation)
- Ground-truth cell-type label of the perturbed type for each simulated dataset
- ScTypeDB-format XLSX marker database with tissueType, cellName, geneSymbolmore1 (positive markers), geneSymbolmore2 (negative markers)
- Scaled/normalized Seurat expression matrix per dataset

## Outputs

- Per-method, per-dataset outcome table recording whether the perturbed cell type is left undetected/unknown across the 45 datasets
- Aggregated unknown-detection counts per method (ScType, SCINA, scSorter)

## How to apply

Load the 45 simulated scRNA-seq count matrices (one per dropout perturbation), each with the ground-truth label of the perturbed cell type. For each dataset, edit the ScTypeDB-format XLSX (columns tissueType, cellName, geneSymbolmore1 positive markers, geneSymbolmore2 negative markers) so exactly one cell type's markers are removed, then pass the perturbed file to gene_sets_prepare() to build positive (gs) and negative (gs2) marker sets. Run sctype_score() on the Seurat-scaled expression matrix (scaled = TRUE) applying the low-confidence rule that assigns 'Unknown' when the top score falls below ncells/4. Rerun SCINA and scSorter on the same 45 datasets with the identical perturbed marker sets, recording per method and dataset whether the perturbed type is left undetected. Aggregate the 45 outcomes per method into a counts table; ScType's score standardizes each gene into a z-score across all cells and multiplies expression by the cell-type specificity (Sit) score, so verify the scaled matrix before scoring.

## Related tools

- **ScType** (Primary annotator being benchmarked: run sctype_score() with positive (gs) and negative (gs2) marker sets after perturbing its marker database) — https://github.com/IanevskiAleksandr/sc-type
- **SCINA** (Comparator method rerun with the identical perturbed marker sets on the 45 simulated datasets)
- **scSorter** (Comparator method rerun with the identical perturbed marker sets on the 45 simulated datasets)
- **Seurat** (Data processing and normalization producing the scaled expression matrix fed to sctype_score())
- **R** (Implementation language; gene_sets_prepare.R and sctype_score_.R are sourced as R functions) — https://github.com/IanevskiAleksandr/sc-type

## Examples

```
gs_list <- gene_sets_prepare("ScTypeDB_perturbed.xlsx", "Immune system"); es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)
```

## Evaluation signals

- Reproduced counts must match the reported unknown-detection rates: ScType 43/45 (95.5%), SCINA 41/45 (91.1%), scSorter 22/45 (48.8%)
- Each of the 45 perturbed marker files must have exactly one cell type's signatures removed (checked against the ScTypeDB-format columns tissueType, cellName, geneSymbolmore1, geneSymbolmore2)
- The low-confidence rule must trigger 'Unknown' only when the top sctype_score value is below ncells/4
- All three methods must be run on the same scaled expression matrix and the identical perturbed marker sets per dataset
- The final counts table must contain one outcome per method-dataset pair (3 methods x 45 datasets = 135 records)

## Limitations

- scCATCH could not be included in this simulated-data comparison because its implementation does not allow custom markers
- Drop-out events are not corrected in the current ScType version; the authors plan to incorporate correction methods such as scDoc to better identify true negative markers
- Negative markers are still relatively few compared to positive markers, which limits annotation performance
- ScType's z-score transformation may be suboptimal for single-cell transcriptomic data

## Evidence

- [methods] load gene set preparation function: "# load gene set preparation function
source("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/gene_sets_prepare.R")"
- [discussion] ScType outperformed comparator methods: "Our unsupervised approach outperformed the prior knowledge-based methods scSorter30, SCINA33, and scCATCH34"
- [discussion] scCATCH excluded from simulated comparison: "the scCATCH method was not used in these comparisons since its implementation does not allow for the use custom markers"
- [discussion] dropout correction not in current version: "we plan to incorporate correction methods for drop-out events in scRNA-seq data, such as scDoc51, which should help to identify true and robust negative"
- [readme] sctype_score call signature from README: "es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)"
