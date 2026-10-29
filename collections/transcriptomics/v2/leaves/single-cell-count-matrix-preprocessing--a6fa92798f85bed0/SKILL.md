---
name: single-cell-count-matrix-preprocessing
description: Use when raw per-individual UMI count matrices from 10x Chromium scRNA-seq
  need gene filtering, top-1000 HVG selection, and marker-based approximated ground-truth
  cell-type labels before clustering benchmarking.
when_to_use_negative:
- The matrices are already filtered and reduced to top 1000 HVGs and only clustering
  is needed — run BAMMSC directly on the prepared list of count matrices instead
- Rare cell type identification is the analysis goal — the authors state that filtering
  genes with excessive zeros may be problematic for rare cell type identification
- No cell-type-specific marker panel (Supplementary Tables 2-4 or equivalent) exists
  for the tissue, so unambiguous marker-based labels cannot be assigned
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BAMM-SC
  role: R package whose BAMMSC/DIMMSC functions consume the preprocessed list of per-individual
    UMI count matrices; downstream clustering and adjustedRandIndex evaluation use
    the preprocessed matrices
  repo: https://github.com/lichen-lab/BAMMSC
- name: 10x Genomics Chromium system
  role: Droplet-based instrument that generated the raw scRNA-seq libraries being
    preprocessed (PBMC, mouse lung, human skin datasets)
- name: Cell Ranger
  role: 10x pipeline used to process libraries; its aggregation downsampling is avoided
    during preprocessing because it equalizes sequencing depth and causes information
    loss
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:13acdb7a88ab674fda5b4b4a06e7f47ea28fea862b08ebc89e3d24cd10ba4ac9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/BAMMSC
  tools:
  - 10x Genomics Chromium system
  - BAMM-SC
  - Cell Ranger
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# single-cell-count-matrix-preprocessing

## Summary

Preprocesses raw droplet-based scRNA-seq UMI count matrices (10x Chromium / Cell Ranger output) by filtering genes expressed in fewer than 1% of cells, selecting the top 1000 highly variable genes, and scoring cell-type marker panels (Supplementary Tables 2-4) to produce approximated ground-truth labels for BAMM-SC clustering benchmarking.

## When to use

Use when raw per-individual UMI count matrices from 10x Chromium scRNA-seq need gene filtering, top-1000 HVG selection, and marker-based approximated ground-truth cell-type labels before clustering benchmarking.

## When NOT to use

- The matrices are already filtered and reduced to top 1000 HVGs and only clustering is needed — run BAMMSC directly on the prepared list of count matrices instead
- Rare cell type identification is the analysis goal — the authors state that filtering genes with excessive zeros may be problematic for rare cell type identification
- No cell-type-specific marker panel (Supplementary Tables 2-4 or equivalent) exists for the tissue, so unambiguous marker-based labels cannot be assigned

## Inputs

- Raw UMI count matrices stored as a list of per-individual gene-by-cell count matrices (genes as rows, cells as columns), generated with the 10x Genomics Chromium system and Cell Ranger
- Cell-type-specific marker gene panels from Supplementary Tables 2-4
- Per-dataset marker-assignment coverage thresholds (PBMC >70%, mouse lung >66%, human skin >67%) for validation

## Outputs

- Labeled cell-by-gene matrices restricted to the top 1000 HVGs, one per dataset (PBMC, mouse lung, skin)
- Marker-based approximated ground-truth cell-type labels with ambiguous cells removed
- Per-dataset marker assignment rates
- Combined artifact saved to marker_labeled_datasets.rds

## How to apply

Load the raw UMI count matrices for each dataset (human PBMC, mouse lung, human skin) as a list of per-individual gene-by-cell count matrices — the format required by BAMMSC's X argument. Filter out genes detected in fewer than 1% of cells, since the paper removed genes with excessive zeros under the assumption that lowly-expressed genes do not contribute much to clustering; note that avoiding Cell Ranger's aggregation downsampling is important because equalizing sequencing depth causes substantial information loss for individuals with higher depth. Select the top 1000 highly variable genes from the filtered matrices to define the clustering feature space. Score the cell-type-specific marker panels (Supplementary Tables 2-4) against every cell, remove cells whose marker signals do not support an unambiguous type, and record per-dataset marker assignment rates. Validate that marker-based labeling coverage exceeds the per-dataset thresholds (>70% for PBMC, >66% for mouse lung, >67% for human skin) so the labels are usable as approximated ground truth for downstream adjusted-Rand-index benchmarking.

## Related tools

- **BAMM-SC** (R package whose BAMMSC/DIMMSC functions consume the preprocessed list of per-individual UMI count matrices; downstream clustering and adjustedRandIndex evaluation use the preprocessed matrices) — https://github.com/lichen-lab/BAMMSC
- **10x Genomics Chromium system** (Droplet-based instrument that generated the raw scRNA-seq libraries being preprocessed (PBMC, mouse lung, human skin datasets))
- **Cell Ranger** (10x pipeline used to process libraries; its aggregation downsampling is avoided during preprocessing because it equalizes sequencing depth and causes information loss)

## Examples

```
install.packages("devtools"); library(devtools); install_github("lichen-lab/BAMMSC"); library(BAMMSC); data("data_BAMMSC"); result=BAMMSC(data_BAMMSC,K=4)
```

## Evaluation signals

- Marker-based labeling coverage exceeds the per-dataset thresholds: >70% for PBMC, >66% for mouse lung, >67% for human skin
- Retained cell-by-gene matrices contain exactly the top 1000 highly variable genes, with identical gene sets across individuals in the list
- Removed cells are only those whose marker signals do not support an unambiguous type; per-dataset assignment rates are recorded
- Output preserves the expected structure: a list of per-individual gene-by-cell matrices where the number of genes is shared but the number of cells may differ across individuals
- Preprocessed matrices and labels support downstream ARI computation via adjustedRandIndex(unlist(Zpred), unlist(Z))

## Limitations

- Filtering genes with excessive zeros may be problematic for rare cell type identification
- The downstream model (BAMM-SC) does not explicitly model a zero-inflation pattern, which may or may not affect clustering accuracy
- Cell Ranger aggregation downsamples reads so all libraries share the same sequencing depth, causing substantial information loss for individuals with higher sequencing depth
- Marker-based labels are only approximated ground truth, not expert-validated annotations

## Evidence

- [discussion] we filtered out genes with excessive zeros from the analysis under the assumption that lowly-expressed genes do not contribute much to clustering: "we filtered out genes with excessive zeros from the analysis under the assumption that lowly-expressed genes do not contribute much to clustering"
- [readme] X: a list with each element a UMI count matrix for each individual, with row as number of genes and column as number of cells.: "X: a list with each element a UMI count matrix for each individual, with row as number of genes and column as number of cells."
- [intro] reads from different libraries are downsampled such that all libraries have the same sequencing depth, leading to substantial information loss: "reads from different libraries are downsampled such that all libraries have the same sequencing depth, leading to substantial information loss"
- [discussion] This may be problematic for rare cell type identification: "This may be problematic for rare cell type identification"
