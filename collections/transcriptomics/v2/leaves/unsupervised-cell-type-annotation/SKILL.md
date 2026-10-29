---
name: unsupervised-cell-type-annotation
description: Use when annotating clusters in a normalized/scaled scRNA-seq matrix
  (e.g., a Seurat object) without a labeled reference, and only a marker database
  plus the expression data are available.
when_to_use_negative:
- Input requires distinguishing malignant from healthy cells — use ScType's single-cell
  SNV calling workflow (vartrix) rather than marker-based annotation alone
- A labeled reference scRNA-seq training dataset is available and a supervised classifier
  (e.g., CaSTLe, ACTINN) is preferred
- The comparator method cannot accept custom markers — as with scCATCH, whose implementation
  does not allow custom marker input
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ScType
  role: 'Core annotation method: gene_sets_prepare builds positive/negative marker
    gene sets and sctype_score assigns cell types'
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: R
  role: Open-source implementation language for the ScType R package
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: Seurat
  role: Pre-processing and clustering of the scRNA-seq data (normalization, PCA, FindClusters)
    before ScType annotation
- name: Splatter
  role: Simulation of scRNA-seq datasets at controlled dropout levels (~50%, ~65%,
    ~80%) for dropout-robustness evaluation
- name: scSorter
  role: Prior-knowledge-based comparator run on the same marker inputs to benchmark
    ScType accuracy and running time
- name: SCINA
  role: Prior-knowledge-based comparator run on the same marker inputs to benchmark
    ScType accuracy and running time
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:a5e98660599d6753a0dfdcfaf5a1d1bb21283b51e71750ebeb1d4f9e372aa8a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - R
  - SCINA
  - ScType
  - Seurat
  - Splatter
  - scSorter
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# unsupervised-cell-type-annotation

## Summary

Annotate cell clusters in a single scRNA-seq dataset using ScType, an unsupervised method that scores clusters against cell-type-specific positive and negative marker gene sets from the ScType marker database, without any reference dataset or manual marker curation.

## When to use

Use when annotating clusters in a normalized/scaled scRNA-seq matrix (e.g., a Seurat object) without a labeled reference, and only a marker database plus the expression data are available.

## When NOT to use

- Input requires distinguishing malignant from healthy cells — use ScType's single-cell SNV calling workflow (vartrix) rather than marker-based annotation alone
- A labeled reference scRNA-seq training dataset is available and a supervised classifier (e.g., CaSTLe, ACTINN) is preferred
- The comparator method cannot accept custom markers — as with scCATCH, whose implementation does not allow custom marker input

## Inputs

- Normalized and scaled scRNA-seq expression matrix or Seurat object (assay = 'RNA', scaled = TRUE)
- ScType marker database file (e.g., ScTypeDB_short.xlsx / ScTypeDB_full.xlsx) with a specified tissue type
- Optional: simulated or reference datasets with known cell-type labels for accuracy evaluation

## Outputs

- Cell-type-by-cell score matrix (es.max) from sctype_score
- Per-cluster/cell cell-type annotations (e.g., 'sctype_classification' column in Seurat metadata via run_sctype)
- Per-dataset annotation accuracy summary (e.g., annotation_accuracy_vs_dropout.csv) when benchmarked against known labels

## How to apply

Normalize and cluster the data (e.g., Seurat: NormalizeData with LogNormalize, ScaleData, RunPCA, FindClusters), then standardize gene expression to z-scores and weight each gene by its cell-type specificity (Sit) score, as ScType does to combine multiple markers. Prepare positive and negative marker gene sets per tissue with the gene_sets_prepare function against the ScType database (e.g., ScTypeDB_short.xlsx or ScTypeDB_full.xlsx, choosing a tissue such as 'Immune system'), then run sctype_score with gs = gs_positive and gs2 = gs_negative to obtain a cell-type-by-cell score matrix. Marker specificity is guaranteed across both cell clusters and cell types, which is what makes annotation unbiased and selective; the method requires neither reference scRNA-seq datasets nor manual marker selection. For robustness checks, annotations can be benchmarked against known labels (e.g., on Splatter-simulated data at ~50/65/80% dropout) and against prior-knowledge comparators such as scSorter and SCINA using the same marker inputs.

## Related tools

- **ScType** (Core annotation method: gene_sets_prepare builds positive/negative marker gene sets and sctype_score assigns cell types) — https://github.com/IanevskiAleksandr/sc-type
- **R** (Open-source implementation language for the ScType R package) — https://github.com/IanevskiAleksandr/sc-type
- **Seurat** (Pre-processing and clustering of the scRNA-seq data (normalization, PCA, FindClusters) before ScType annotation)
- **Splatter** (Simulation of scRNA-seq datasets at controlled dropout levels (~50%, ~65%, ~80%) for dropout-robustness evaluation)
- **scSorter** (Prior-knowledge-based comparator run on the same marker inputs to benchmark ScType accuracy and running time)
- **SCINA** (Prior-knowledge-based comparator run on the same marker inputs to benchmark ScType accuracy and running time)

## Examples

```
source('https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/gene_sets_prepare.R'); source('https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/sctype_score_.R'); gs <- gene_sets_prepare('ScTypeDB_short.xlsx','Immune system'); es.max <- sctype_score(scRNAseqData=scRNAseqData, scaled=TRUE, gs=gs$gs_positive, gs2=gs$gs_negative)
```

## Evaluation signals

- Annotation accuracy against known cell-type labels exceeds 90% even at ~80% dropout in Splatter-simulated datasets (15 datasets per level at ~50/65/80% dropout)
- ScType accuracy and running time outperform scSorter and SCINA on the same marker inputs
- sctype_score returns a cell-type-by-cell score matrix (es.max) with the expected dimensions (n cell types x n cells) for the chosen tissue's gene sets
- Annotations are selective because marker specificity is enforced across both cell clusters and cell types; unassigned/ambiguous clusters indicate overly permissive marker sets
- Aggregated accuracy-by-dropout results reproduce the expected monotone-free behavior (accuracy stays >90% from ~50% to ~80% dropout)

## Limitations

- Drop-out events in scRNA-seq data are not corrected in the current version; incorporating correction methods such as scDoc is planned to identify true and robust negative markers
- Z-score transformation of gene expression, used for computationally-efficient combination of multiple markers, may be suboptimal for single-cell transcriptomic data
- Negative markers are still relatively few compared to positive markers, limiting annotation; results are expected to improve as more negative markers are curated

## Evidence

- [discussion] Z-score specificity scoring: "ScType first standardizes each gene expression profile into z-score across all cells."
- [discussion] No reference needed: "ScType requires neither reference scRNA-seq datasets nor manual selection of marker genes"
- [discussion] Outperforms comparator methods: "Our unsupervised approach outperformed the prior knowledge-based methods scSorter30, SCINA33, and scCATCH34"
- [results] Dropout robustness: [evidence span withheld — anchor status: not_found]
- [readme] README quick start: "gs_list <- gene_sets_prepare("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/ScTypeDB_short.xlsx", "Immune system")"
- [methods] Scoring call: "es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)"
