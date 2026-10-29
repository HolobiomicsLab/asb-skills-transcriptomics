---
name: gene-expression-filtering
description: Use when raw gene-by-cell UMI count matrices from multiple individuals
  (e.g., human PBMC, mouse lung, human skin 10x Chromium data) must be filtered of
  lowly-expressed genes and reduced to a top-1000-HVG feature space before clustering
  or benchmarking.
when_to_use_negative:
- The goal is rare cell type identification — the authors note that filtering out
  genes with excessive zeros 'may be problematic for rare cell type identification'
  because lowly-expressed genes specific to rare populations are discarded
- The input is already a filtered/feature-selected expression matrix (e.g., already
  restricted to HVGs) rather than raw UMI counts
- The analysis requires genes expressed in <1% of cells (e.g., rare-population markers)
  to be retained for interpretation rather than removed
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: BAMM-SC
  role: Downstream Bayesian mixture-model clustering (R package BAMMSC) that consumes
    the filtered gene-by-cell UMI count matrices; input X must be a list of gene-by-cell
    count matrices, one per individual
  repo: github.com/lichen-lab/BAMMSC
- name: Cell Ranger
  role: Upstream 10x Genomics pipeline that aggregates multiple libraries into the
    raw count matrices; its downsampling-based depth equalization motivates preprocessing
    choices for multi-individual data
- name: 10x Genomics Chromium system
  role: Droplet-based platform that generated the scRNA-seq data (e.g., PBMCs from
    4 healthy donors) whose count matrices this filtering skill prepares
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:bf684cd4a75e84b417f77f705a6fa7699851fe3831df5dfc5405c00d99c0f797
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 10x Genomics Chromium system
  - BAMM-SC
  - Cell Ranger
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# gene-expression-filtering

## Summary

Preprocesses raw droplet-based scRNA-seq UMI count matrices by removing genes with excessive zeros (detected in fewer than 1% of cells) and restricting the feature space to the top 1000 highly variable genes, so that downstream clustering (e.g., BAMM-SC) and marker-based ground-truth labeling operate on an informative gene set.

## When to use

Use when raw gene-by-cell UMI count matrices from multiple individuals (e.g., human PBMC, mouse lung, human skin 10x Chromium data) must be filtered of lowly-expressed genes and reduced to a top-1000-HVG feature space before clustering or benchmarking.

## When NOT to use

- The goal is rare cell type identification — the authors note that filtering out genes with excessive zeros 'may be problematic for rare cell type identification' because lowly-expressed genes specific to rare populations are discarded
- The input is already a filtered/feature-selected expression matrix (e.g., already restricted to HVGs) rather than raw UMI counts
- The analysis requires genes expressed in <1% of cells (e.g., rare-population markers) to be retained for interpretation rather than removed

## Inputs

- Raw UMI count matrices for droplet-based scRNA-seq datasets (human PBMC from 5 pooled donors, mouse lung, human skin), each as a list of per-individual gene-by-cell count matrices
- Cell-type-specific marker gene panels (Supplementary Tables 2-4 of the BAMM-SC paper) for optional marker-based labeling

## Outputs

- Filtered cell-by-gene count matrices restricted to the top 1000 highly variable genes, with marker-based approximated ground-truth cell-type labels
- Per-dataset marker assignment rates (e.g., saved together as marker_labeled_datasets.rds for PBMC, mouse lung, and skin)

## How to apply

Load the raw UMI count matrices for each dataset as a list of per-individual gene-by-cell count matrices (genes in rows, cells in columns, as required by the BAMMSC R package). For each dataset, filter out genes detected in fewer than 1% of cells — the authors' rationale is that lowly-expressed genes contribute little to clustering. From the filtered matrices, select the top 1000 highly variable genes to define the clustering feature space. Optionally score cell-type-specific marker panels (Supplementary Tables 2-4) against every cell to obtain marker-based approximated ground-truth labels, removing cells whose marker signals do not support an unambiguous type. Validate that marker-based labeling coverage exceeds per-dataset thresholds (>70% for PBMC, >66% for mouse lung, >67% for human skin) before using the labels for adjusted-Rand-index benchmarking.

## Related tools

- **BAMM-SC** (Downstream Bayesian mixture-model clustering (R package BAMMSC) that consumes the filtered gene-by-cell UMI count matrices; input X must be a list of gene-by-cell count matrices, one per individual) — github.com/lichen-lab/BAMMSC
- **Cell Ranger** (Upstream 10x Genomics pipeline that aggregates multiple libraries into the raw count matrices; its downsampling-based depth equalization motivates preprocessing choices for multi-individual data)
- **10x Genomics Chromium system** (Droplet-based platform that generated the scRNA-seq data (e.g., PBMCs from 4 healthy donors) whose count matrices this filtering skill prepares)

## Evaluation signals

- Every retained gene is detected in at least 1% of cells (genes expressed in fewer than 1% of cells are removed)
- The resulting feature space contains exactly the top 1000 highly variable genes per dataset
- Matrix orientation check: each per-individual element is a gene-by-cell count matrix, with identical gene sets across individuals but potentially differing cell counts
- Marker-based labeling coverage exceeds per-dataset thresholds: >70% for PBMC, >66% for mouse lung, >67% for human skin
- Downstream sanity check: clustering of the filtered matrices can be scored against the approximated ground-truth labels via adjustedRandIndex

## Limitations

- Filtering genes with excessive zeros may be problematic for rare cell type identification
- Filtering assumes lowly-expressed genes do not contribute much to clustering, which may not hold for all tissues
- Cells whose marker signals do not support an unambiguous cell type must be removed, so label coverage is dataset-dependent and must be re-checked per dataset

## Evidence

- [discussion] Rationale for zero-gene filtering: "we filtered out genes with excessive zeros from the analysis under the assumption that lowly-expressed genes do not contribute much to clustering"
- [discussion] Limitation of zero-gene filtering: "This may be problematic for rare cell type identification"
- [readme] Required input format for the filtered matrices: "X: a list with each element a UMI count matrix for each individual, with row as number of genes and column as number of cells."
