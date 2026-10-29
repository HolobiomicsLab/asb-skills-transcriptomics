---
name: unknown-cell-type-detection-assessment
description: Use when evaluating whether an annotation method correctly reports 'Unknown'
  for cell types whose marker signatures are absent, via leave-one-cell-type-out tests
  on simulated scRNA-seq count matrices with known ground truth.
when_to_use_negative:
- The goal is routine annotation of a well-characterized dataset with an intact marker
  database — this skill is a perturbation benchmark, not a production annotation workflow
- The method under test cannot accept custom/perturbed marker sets (e.g., scCATCH
  was excluded from these comparisons 'since its implementation does not allow for
  the use custom markers')
- Input data are already cell-type-labeled reference datasets for supervised classifiers
  (e.g., CaSTLe, ACTINN), since this skill targets prior-knowledge marker-based unsupervised
  methods
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: ScType
  role: Primary method under test; cell-type scoring via sctype_score() with positive
    (gs) and negative (gs2) marker sets and an 'Unknown' low-confidence rule (top
    score below ncells/4)
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: SCINA
  role: Comparison method rerun with identical perturbed marker sets to measure unknown-detection
    rate (41/45)
- name: scSorter
  role: Comparison method rerun with identical perturbed marker sets; weakest at leaving
    the perturbed type undetected (22/45)
- name: Seurat
  role: Data processing and normalization of the scRNA-seq matrices (NormalizeData,
    ScaleData) prior to scoring
- name: R
  role: Implementation environment; ScType functions sourced from the sc-type GitHub
    repository
  repo: https://github.com/IanevskiAleksandr/sc-type
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:6cbe2283854919e18676a0a8f1c6e535881cf03106dfd2af7bb916fd078bd0a1
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# unknown-cell-type-detection-assessment

## Summary

A benchmarking skill for leave-one-cell-type-out evaluation of marker-based scRNA-seq annotation methods (ScType, SCINA, scSorter): perturb a marker database by removing one cell type's signatures at a time, rerun each method, and quantify how reliably the perturbed cell type is left undetected/unknown. It probes whether annotation methods exploit marker specificity (positive and negative markers) rather than merely assigning the best-scoring label.

## When to use

Use when evaluating whether an annotation method correctly reports 'Unknown' for cell types whose marker signatures are absent, via leave-one-cell-type-out tests on simulated scRNA-seq count matrices with known ground truth.

## When NOT to use

- The goal is routine annotation of a well-characterized dataset with an intact marker database — this skill is a perturbation benchmark, not a production annotation workflow
- The method under test cannot accept custom/perturbed marker sets (e.g., scCATCH was excluded from these comparisons 'since its implementation does not allow for the use custom markers')
- Input data are already cell-type-labeled reference datasets for supervised classifiers (e.g., CaSTLe, ACTINN), since this skill targets prior-knowledge marker-based unsupervised methods

## Inputs

- 45 simulated scRNA-seq count matrices (one per perturbation) from dropout simulation
- Ground-truth cell-type label of the perturbed type for each of the 45 datasets
- ScTypeDB-format XLSX marker database with tissueType, cellName, geneSymbolmore1 (positive markers), and geneSymbolmore2 (negative markers) columns
- Scaled/normalized (z-score) scRNA-seq expression matrix per dataset (e.g., Seurat ScaleData output)

## Outputs

- Per-dataset, per-method binary outcome of whether the perturbed cell type is left undetected/unknown
- Aggregated counts table over 45 datasets per method (ScType, SCINA, scSorter) with unknown-detection counts and rates

## How to apply

Start from 45 simulated scRNA-seq count matrices (from dropout simulation), each with the ground-truth label of the perturbed cell type. For each dataset, edit an ScTypeDB-format XLSX marker database (columns tissueType, cellName, geneSymbolmore1 positive markers, geneSymbolmore2 negative markers) so that exactly one cell type's marker signatures are removed, then load the perturbed sets with gene_sets_prepare() and run sctype_score() on the scaled (z-score) expression matrix with positive (gs) and negative (gs2) marker sets. Apply ScType's low-confidence rule assigning 'Unknown' when the top score is below ncells/4. Rerun SCINA and scSorter with the identical perturbed marker sets on the same 45 datasets, recording per method and dataset whether the perturbed type goes undetected, then aggregate counts per method. Validate by comparing reproduced counts to reported rates: ScType 43/45 (95.5%), SCINA 41/45 (91.1%), scSorter 22/45 (48.8%). The rationale is that ScType's use of marker specificity across both clusters and cell types, plus negative markers, makes it better at withholding a label when markers are missing.

## Related tools

- **ScType** (Primary method under test; cell-type scoring via sctype_score() with positive (gs) and negative (gs2) marker sets and an 'Unknown' low-confidence rule (top score below ncells/4)) — https://github.com/IanevskiAleksandr/sc-type
- **SCINA** (Comparison method rerun with identical perturbed marker sets to measure unknown-detection rate (41/45))
- **scSorter** (Comparison method rerun with identical perturbed marker sets; weakest at leaving the perturbed type undetected (22/45))
- **Seurat** (Data processing and normalization of the scRNA-seq matrices (NormalizeData, ScaleData) prior to scoring)
- **R** (Implementation environment; ScType functions sourced from the sc-type GitHub repository) — https://github.com/IanevskiAleksandr/sc-type

## Examples

```
gs_list <- gene_sets_prepare("ScTypeDB_perturbed.xlsx", "Immune system"); es.max <- sctype_score(scRNAseqData = scaled_expr, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)
```

## Evaluation signals

- Reproduced ScType unknown-detection count equals 43/45 (95.5%) across the 45 simulated datasets
- Reproduced SCINA count equals 41/45 (91.1%) and scSorter equals 22/45 (48.8%) on the same datasets
- For every dataset, the marker database passed to gene_sets_prepare() contains exactly one removed cell type (geneSymbolmore1/geneSymbolmore2 rows for that cellName absent)
- ScType assigns 'Unknown' only when the top sctype_score value is below ncells/4 (low-confidence rule applied consistently)
- All three methods consume the identical scaled expression matrix and identical perturbed marker sets per dataset (paired design)

## Limitations

- scCATCH cannot participate because 'its implementation does not allow for the use custom markers', limiting the comparison to ScType, SCINA, and scSorter
- ScType relies on z-score transformation, 'which may be suboptimal when dealing with single-cell transcriptomic data', so results depend on the scaling choice
- Drop-out events in scRNA-seq data are not corrected in the current ScType version, which may affect negative-marker evidence in simulated sparse data
- Negative markers remain relatively few compared to positive markers, limiting the strength of evidence-against signals in the unknown-detection test

## Evidence

- [methods] Comparison methods use identical perturbed marker sets and the ScType scoring pipeline: "# load gene set preparation function
source("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/gene_sets_prepare.R")"
- [methods] ScType cell-type annotation scoring function used on the scaled matrix: "# load cell type annotation function
source("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/sctype_score_.R")"
- [discussion] ScType outperformed the marker-based comparison methods included in the leave-one-type-out test: "Our unsupervised approach outperformed the prior knowledge-based methods scSorter30, SCINA33, and scCATCH34"
- [discussion] Positive and negative markers are combined via z-scored expression and a specificity score: [evidence span withheld — anchor status: not_found]
- [abstract] ScType enables fully-automated, unsupervised annotation from a single dataset plus a marker database: "we developed a computational platform, ScType, which enables a fully-automated and ultra-fast cell-type identification"
- [readme] README quick start defines the gs/gs2 scoring call used in the benchmark: "es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)"
