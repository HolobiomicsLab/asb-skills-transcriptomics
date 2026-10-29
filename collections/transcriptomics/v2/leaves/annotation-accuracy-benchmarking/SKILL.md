---
name: annotation-accuracy-benchmarking
description: Use when you need to quantify how robust an automated cell-type annotation
  method is to dropout events, by benchmarking accuracy against known labels on simulated
  scRNA-seq data at graded dropout levels.
when_to_use_negative:
- Your input is real experimental scRNA-seq data without ground-truth cell-type labels
  — accuracy cannot be computed, so use manual/curation-based validation instead
- You need dropout correction rather than robustness measurement — the current ScType
  version does not correct drop-out events (the authors plan to incorporate scDoc)
- You want to include scCATCH as a comparator on simulated data — its implementation
  does not allow custom markers
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Splatter
  role: Simulates the 45 scRNA-seq datasets with controlled dropout rates and known
    cell-type labels for ground-truth benchmarking
- name: ScType
  role: Primary annotation method under test; gene_sets_prepare builds positive/negative
    marker sets and sctype_score assigns cell types
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: scSorter
  role: Comparator prior knowledge-based annotation method run on the same simulated
    datasets and marker inputs
- name: SCINA
  role: Comparator prior knowledge-based annotation method run on the same simulated
    datasets and marker inputs
- name: R
  role: Implementation environment for simulation, annotation, and accuracy aggregation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:59edc50127b03eace99f2f4b84b7361debca1df99cfb07931b64b1db90a7ef9e
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# annotation-accuracy-benchmarking

## Summary

Benchmark automated cell-type annotation accuracy under controlled scRNA-seq dropout stress by simulating 45 Splatter datasets (~50%, ~65%, ~80% dropout) with known labels, running ScType, scSorter, and SCINA on identical marker inputs, and scoring per-dataset accuracy against ground truth.

## When to use

Use when you need to quantify how robust an automated cell-type annotation method is to dropout events, by benchmarking accuracy against known labels on simulated scRNA-seq data at graded dropout levels.

## When NOT to use

- Your input is real experimental scRNA-seq data without ground-truth cell-type labels — accuracy cannot be computed, so use manual/curation-based validation instead
- You need dropout correction rather than robustness measurement — the current ScType version does not correct drop-out events (the authors plan to incorporate scDoc)
- You want to include scCATCH as a comparator on simulated data — its implementation does not allow custom markers

## Inputs

- Splatter simulation parameters: 45 datasets, 3 dropout levels (~50%, ~65%, ~80%), dropout.shape {-0.5, 0, 2.5}, dropout.mid {-2.5, 0, 2.5}, 10 cell types, scaled lognormal proportions, de.facScale {0.5, 0.6, 0.7}
- Per-cell-type marker gene sets (ten highly specific and ten low-specificity markers from Wilcoxon rank-sum DE)
- ScType marker database file ScTypeDB_full.xlsx
- Known simulated cell-type labels per dataset

## Outputs

- Per-dataset annotation accuracy values for ScType, scSorter, and SCINA
- Accuracy summary aggregated by method and dropout level, written to annotation_accuracy_vs_dropout.csv
- Running-time comparison across the three methods

## How to apply

Simulate 45 scRNA-seq datasets with Splatter in R — 15 per dropout level (~50%, ~65%, ~80% of genes dropped out) — sweeping dropout.shape over {-0.5, 0, 2.5} and dropout.mid over {-2.5, 0, 2.5}, each dataset with 10 cell types at scaled lognormal proportions and de.facScale drawn from {0.5, 0.6, 0.7}. For each dataset, perform Wilcoxon rank-sum differential expression between cell types to select ten highly specific and ten low-specificity marker genes per cell type, then prepare positive (gs) and negative (gs2) gene sets with the ScType gene_sets_prepare function against ScTypeDB_full.xlsx. Run ScType (sctype_score with gs and gs2) plus the comparators scSorter and SCINA on the same datasets using the same marker inputs, and compute per-dataset annotation accuracy against the simulated cell-type labels, aggregating by method and dropout level. The rationale is that simulated data carry ground-truth labels, letting you isolate the effect of dropout — a limitation the ScType authors explicitly flag as uncorrected in their method — on annotation performance relative to other prior knowledge-based methods.

## Related tools

- **Splatter** (Simulates the 45 scRNA-seq datasets with controlled dropout rates and known cell-type labels for ground-truth benchmarking)
- **ScType** (Primary annotation method under test; gene_sets_prepare builds positive/negative marker sets and sctype_score assigns cell types) — https://github.com/IanevskiAleksandr/sc-type
- **scSorter** (Comparator prior knowledge-based annotation method run on the same simulated datasets and marker inputs)
- **SCINA** (Comparator prior knowledge-based annotation method run on the same simulated datasets and marker inputs)
- **R** (Implementation environment for simulation, annotation, and accuracy aggregation)

## Examples

```
gs_list <- gene_sets_prepare("ScTypeDB_full.xlsx", tissue); es.max <- sctype_score(scRNAseqData = sim_counts, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)
```

## Evaluation signals

- ScType annotation accuracy stays above 90% even at the ~80% dropout level
- annotation_accuracy_vs_dropout.csv exists and contains accuracy per method and dropout level for all 45 datasets
- ScType accuracy generally exceeds scSorter and SCINA across dropout levels when all methods receive the same marker inputs
- Each of the 45 simulated datasets carries known cell-type labels enabling exact accuracy computation (correct calls / total cells)
- scSorter and SCINA runs use identical marker gene sets as ScType, ensuring a fair comparator setup

## Limitations

- Drop-out events in scRNA-seq data are not corrected in the current ScType version; the authors plan to incorporate correction methods such as scDoc to identify true and robust negative markers
- Results depend on the simulation regime (Splatter parameter ranges); conclusions may not transfer to all tissue types or real dropout distributions
- scCATCH could not be included in the simulated-data comparison because its implementation does not allow custom markers

## Evidence

- [results] ScType robustness at high dropout: [evidence span withheld — anchor status: not_found]
- [discussion] ScType outperforms comparators: "Our unsupervised approach outperformed the prior knowledge-based methods scSorter30, SCINA33, and scCATCH34"
- [discussion] Dropout not corrected: "we plan to incorporate correction methods for drop-out events in scRNA-seq data, such as scDoc51, which should help to identify true and robust negative markers"
- [methods] ScType functions used in workflow: "source("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/gene_sets_prepare.R")"
- [readme] ScType quick-start invocation: "es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)"
