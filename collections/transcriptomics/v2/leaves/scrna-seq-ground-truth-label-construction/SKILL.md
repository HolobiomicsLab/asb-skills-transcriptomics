---
name: scrna-seq-ground-truth-label-construction
description: Use when real per-individual UMI count matrices (e.g., 10x Chromium PBMC,
  mouse lung, skin) lack definitive cell-type labels and you need marker-derived approximated
  ground truth to benchmark clustering via adjusted Rand index.
when_to_use_negative:
- Simulated or synthetic datasets where genuine cell labels already exist (e.g., the
  BAMMSC example data data_BAMMSC with known z = rep(seq(K),each=100)) — no marker-based
  approximation is needed
- Analyses targeting rare cell types, since the paper notes that filtering genes with
  excessive zeros may be problematic for rare cell type identification
- Clustering runs where no cell-type marker panel is available for the tissue, leaving
  no basis for approximated ground-truth assignment
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: BAMM-SC
  role: Multi-individual Bayesian mixture clustering method whose predictions are
    benchmarked against these marker-derived approximated ground-truth labels via
    adjustedRandIndex
  repo: https://github.com/lichen-lab/BAMMSC
- name: 10x Genomics Chromium system
  role: Droplet-based platform that generated the per-donor scRNA-seq libraries whose
    UMI count matrices are labeled by this skill
- name: Cell Ranger
  role: 10x pipeline that processes the raw libraries into per-individual gene-by-cell
    count matrices used as input; its aggregation downsampling is a known information-loss
    caveat
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:5cfa0d8cb4f09a22b04b6776c97b83280f432cacc0534eaf4744a49787a4b7a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/BAMMSC
  tools:
  - 10x Genomics Chromium system
  - BAMM-SC
  - Cell Ranger
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# scRNA-seq-ground-truth-label-construction

## Summary

Constructs approximated ground-truth cell-type labels for real droplet scRNA-seq datasets (human PBMC, mouse lung, human skin) by scoring cell-type-specific marker panels against every cell on a top-1000-HVG feature space, enabling adjusted-Rand-index benchmarking of multi-individual clustering methods such as BAMM-SC.

## When to use

Use when real per-individual UMI count matrices (e.g., 10x Chromium PBMC, mouse lung, skin) lack definitive cell-type labels and you need marker-derived approximated ground truth to benchmark clustering via adjusted Rand index.

## When NOT to use

- Simulated or synthetic datasets where genuine cell labels already exist (e.g., the BAMMSC example data data_BAMMSC with known z = rep(seq(K),each=100)) — no marker-based approximation is needed
- Analyses targeting rare cell types, since the paper notes that filtering genes with excessive zeros may be problematic for rare cell type identification
- Clustering runs where no cell-type marker panel is available for the tissue, leaving no basis for approximated ground-truth assignment

## Inputs

- List of per-individual gene-by-cell UMI count matrices for each real dataset (human PBMC from five pooled donors, mouse lung, human skin)
- Cell-type-specific marker panels per tissue (Supplementary Tables 2-4)

## Outputs

- marker_labeled_datasets.rds containing the labeled cell-by-gene matrices restricted to the top 1000 highly variable genes for PBMC, mouse lung, and skin
- Per-dataset marker-based cell-type assignment rates (PBMC, mouse lung, human skin)

## How to apply

Load the raw UMI count matrices for each real dataset as a list of per-individual gene-by-cell matrices (e.g., pooled PBMC cells from five donors, including one published 10x Genomics donor). Filter out genes detected in fewer than 1% of cells — lowly-expressed genes with excessive zeros are assumed not to contribute much to clustering — then select the top 1000 highly variable genes to define the clustering feature space. Score the cell-type-specific marker panels (Supplementary Tables 2-4) against every cell to obtain marker-based type assignments, and remove cells whose marker signals do not support an unambiguous type. Record the per-dataset marker assignment rates and save the labeled cell-by-gene matrices (top 1000 HVGs) and rates to marker_labeled_datasets.rds. Validate that marker-based labeling coverage exceeds the per-dataset thresholds (>70% PBMC, >66% mouse lung, >67% human skin) before using the labels as approximated ground truth for downstream adjustedRandIndex comparisons.

## Related tools

- **BAMM-SC** (Multi-individual Bayesian mixture clustering method whose predictions are benchmarked against these marker-derived approximated ground-truth labels via adjustedRandIndex) — https://github.com/lichen-lab/BAMMSC
- **10x Genomics Chromium system** (Droplet-based platform that generated the per-donor scRNA-seq libraries whose UMI count matrices are labeled by this skill)
- **Cell Ranger** (10x pipeline that processes the raw libraries into per-individual gene-by-cell count matrices used as input; its aggregation downsampling is a known information-loss caveat)

## Examples

```
library(mclust); data("data_BAMMSC"); Zpred=BAMMSC.predict(X=data_BAMMSC,K=4,p=0.5,seed=1); adjustedRandIndex(unlist(Zpred),unlist(Z))
```

## Evaluation signals

- Marker-based labeling coverage exceeds per-dataset thresholds: >70% for human PBMC, >66% for mouse lung, >67% for human skin
- Each saved cell-by-gene matrix contains exactly the top 1000 highly variable genes after the <1% detection-frequency gene filter
- Cells retained in the output each have an unambiguous marker-based type; cells with ambiguous marker signals are removed and counted in the per-dataset assignment rates
- marker_labeled_datasets.rds parses in R and contains both the labeled matrices and the per-dataset assignment rates for all three tissues
- Downstream benchmarking yields adjusted Rand index values computed with adjustedRandIndex(unlist(predictions), unlist(marker-derived labels)) for BAMM-SC versus SIMLR, CellTree, SC3, TSCAN, and DIMM-SC

## Limitations

- Filtering out genes with excessive zeros assumes lowly-expressed genes do not contribute to clustering but may be problematic for rare cell type identification
- Marker-derived labels are only approximated ground truth; coverage is incomplete because cells with ambiguous marker signals are discarded
- If per-dataset assignment coverage falls below the validation thresholds (>70% PBMC, >66% mouse lung, >67% human skin), the labels are not usable as reliable ground truth for ARI benchmarking

## Evidence

- [discussion] Filtered gene selection rationale for the feature space: "we filtered out genes with excessive zeros from the analysis under the assumption that lowly-expressed genes do not contribute much to clustering"
- [readme] Input format for per-individual count matrices: "X: a list with each element a UMI count matrix for each individual, with row as number of genes and column as number of cells."
- [intro] Precedent for supervised/marker-based cross-dataset type assignment: "Supervised methods, such as MetaNeighbor, have been proposed to assess how well cell-type-specific transcriptional profiles replicate across different datasets"
- [readme] ARI evaluation against constructed reference labels: "adjustedRandIndex(unlist(Zpred),unlist(Z))"
