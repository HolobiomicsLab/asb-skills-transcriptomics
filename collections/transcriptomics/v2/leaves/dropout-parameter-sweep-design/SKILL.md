---
name: dropout-parameter-sweep-design
description: Use when you must quantify how robust a cell-type annotation method is
  to dropout by sweeping Splatter's dropout.shape/dropout.mid parameters over simulated
  datasets with known ground-truth labels.
when_to_use_negative:
- Your input is real experimental scRNA-seq data without ground-truth labels — this
  skill requires simulated datasets with known cell-type labels for accuracy scoring
- The comparator method cannot accept custom marker sets (e.g. scCATCH was excluded
  because 'its implementation does not allow for the use custom markers')
- Your goal is dropout correction/imputation of data rather than benchmarking annotation
  robustness — the article notes dropout correction (e.g. scDoc) is planned but not
  part of the current ScType pipeline
tools:
- name: Splatter
  role: Simulates the 45 scRNA-seq datasets under the dropout.shape/dropout.mid parameter
    sweep with known cell-type labels
- name: R
  role: Language for the Splatter simulation, Wilcorank rank-sum marker selection,
    and accuracy aggregation
- name: ScType
  role: Primary annotation method evaluated for dropout robustness via gene_sets_prepare
    and sctype_score (positive gs, negative gs2)
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: scSorter
  role: Comparator prior-knowledge annotation method run on the same simulated datasets
    and markers
- name: SCINA
  role: Comparator prior-knowledge annotation method run on the same simulated datasets
    and markers
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:f4c00f3a5080dc5b27b5036d9adcad66ce5f245f59fa577853fbad8d622bb164
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - R
  - SCINA
  - ScType
  - Splatter
  - scSorter
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# dropout-parameter-sweep-design

## Summary

Design a Splatter-based simulation parameter sweep that generates scRNA-seq datasets across controlled dropout levels (~50%, ~65%, ~80%) to benchmark the robustness of ScType's marker-based cell-type annotation against comparators such as scSorter and SCINA.

## When to use

Use when you must quantify how robust a cell-type annotation method is to dropout by sweeping Splatter's dropout.shape/dropout.mid parameters over simulated datasets with known ground-truth labels.

## When NOT to use

- Your input is real experimental scRNA-seq data without ground-truth labels — this skill requires simulated datasets with known cell-type labels for accuracy scoring
- The comparator method cannot accept custom marker sets (e.g. scCATCH was excluded because 'its implementation does not allow for the use custom markers')
- Your goal is dropout correction/imputation of data rather than benchmarking annotation robustness — the article notes dropout correction (e.g. scDoc) is planned but not part of the current ScType pipeline

## Inputs

- Splatter simulation parameter grids (dropout.shape ∈ {-0.5, 0, 2.5}; dropout.mid ∈ {-2.5, 0, 2.5}; de.facScale ∈ {0.5, 0.6, 0.7})
- Known simulated cell-type labels (10 cell types per dataset, scaled lognormal proportions)
- ScTypeDB_full.xlsx marker database
- Seurat/normalized count matrices from the 45 simulated datasets

## Outputs

- 45 Splatter-simulated scRNA-seq count matrices (15 per dropout level)
- Per-cell-type positive and negative marker gene sets (10 highly specific + 10 low-specificity markers per cell type)
- annotation_accuracy_vs_dropout.csv: per-dataset annotation accuracy aggregated by method and dropout level

## How to apply

Simulate 45 scRNA-seq datasets in R with Splatter — 15 per dropout level (~50%, ~65%, ~80% of genes dropped out) — by sweeping dropout.shape over {-0.5, 0, 2.5} and dropout.mid over {-2.5, 0, 2.5}, each with 10 cell types at scaled lognormal proportions and de.facScale drawn from {0.5, 0.6, 0.7}. For each dataset, run Wilcoxon rank-sum differential expression between cell types to select ten highly specific and ten low-specificity marker genes per cell type. Prepare positive and negative gene sets with ScType's gene_sets_prepare against the ScTypeDB_full.xlsx marker database, then annotate each dataset with sctype_score (gs = positive markers, gs2 = negative markers) alongside scSorter and SCINA on identical marker inputs. Compute per-dataset accuracy against the known simulated labels and aggregate by method and dropout level to produce a method-by-dropout accuracy summary.

## Related tools

- **Splatter** (Simulates the 45 scRNA-seq datasets under the dropout.shape/dropout.mid parameter sweep with known cell-type labels)
- **R** (Language for the Splatter simulation, Wilcorank rank-sum marker selection, and accuracy aggregation)
- **ScType** (Primary annotation method evaluated for dropout robustness via gene_sets_prepare and sctype_score (positive gs, negative gs2)) — https://github.com/IanevskiAleksandr/sc-type
- **scSorter** (Comparator prior-knowledge annotation method run on the same simulated datasets and markers)
- **SCINA** (Comparator prior-knowledge annotation method run on the same simulated datasets and markers)

## Examples

```
source("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/gene_sets_prepare.R"); source("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/sctype_score_.R"); gs_list <- gene_sets_prepare("ScTypeDB_full.xlsx", tissue_type); es.max <- sctype_score(scRNAseqData = sim_counts, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)
```

## Evaluation signals

- Exactly 45 simulated datasets are produced: 15 at each of the ~50%, ~65%, and ~80% dropout levels
- Each cell type contributes exactly ten highly specific and ten low-specificity marker genes selected by Wilcoxon rank-sum DE
- ScType annotation accuracy remains above 90% even at the ~80% dropout level
- ScType accuracy exceeds scSorter and SCINA at each dropout level, in both accuracy and running time
- The final summary is written to annotation_accuracy_vs_dropout.csv with per-dataset accuracies keyed by method and dropout level

## Limitations

- Drop-out events in scRNA-seq data are not corrected in the current ScType version; correction methods such as scDoc were only planned
- Z-score transformation of gene expression may be suboptimal for single-cell transcriptomic data, potentially affecting scores under extreme dropout
- Findings are grounded in Splatter-simulated data with a specific parameter grid; extrapolation to other simulation settings or real dropout regimes is not validated
- Negative markers remain relatively few compared to positive markers, which can limit annotation performance

## Evidence

- [discussion] We plan to incorporate correction methods for drop-out events: "we plan to incorporate correction methods for drop-out events in scRNA-seq data, such as scDoc51, which should help to identify true and robust negative markers"
- [methods] ScType annotation pipeline functions: "source("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/gene_sets_prepare.R")"
- [readme] ScType scoring call: "es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)"
- [abstract] Validation scope on real datasets: "Using six scRNA-seq datasets from various human and mouse tissues, we show how ScType provides unbiased and accurate cell type annotations"
- [discussion] Z-score transformation caveat: "ScType utilizes z-score transformation for computationally-efficient combination of multiple markers, which may be suboptimal when dealing with single-cell transcriptomic data."
