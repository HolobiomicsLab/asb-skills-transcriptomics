---
name: highly-variable-gene-selection
description: Use when preparing raw per-individual UMI count matrices (e.g., human
  PBMC, mouse lung, human skin droplet-based datasets) for clustering with BAMM-SC
  or competitors, and a reduced gene feature space must be defined before clustering.
when_to_use_negative:
- The goal is rare cell type identification — the article warns that filtering genes
  with excessive zeros 'may be problematic for rare cell type identification'
- The input is already a normalized or reduced feature matrix rather than raw droplet-based
  UMI counts, so re-filtering by zero fraction would double-filter
- The data are bulk RNA-seq or sequencing-depth-downsampled Cell Ranger aggregates,
  since downsampling in the aggregation step causes substantial information loss for
  individuals with higher sequencing depth
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: BAMM-SC
  role: Downstream consumer of the top-1000-HVG count matrices; takes the list of
    per-individual UMI count matrices as X and clusters across individuals
  repo: https://github.com/lichen-lab/BAMMSC
- name: Cell Ranger
  role: Upstream 10x Genomics pipeline that produces the per-library UMI count matrices
    aggregated across individuals before filtering and HVG selection
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:2761b0c34362b80e11998bb1b5ae7385a9572f19ea128d847a14c5d89628681a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/BAMMSC
  tools:
  - 10x Genomics Chromium system
  - BAMM-SC
  - Cell Ranger
  - CellTree
  - DIMM-SC
  - SC3
  - SIMLR
  - TSCAN
  - scran
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# highly-variable-gene-selection

## Summary

Preprocess droplet-based scRNA-seq UMI count matrices by filtering out genes with excessive zeros (detected in fewer than 1% of cells) and retaining the top 1000 highly variable genes, defining a compact feature space consumed by BAMM-SC and competing clustering methods.

## When to use

Use when preparing raw per-individual UMI count matrices (e.g., human PBMC, mouse lung, human skin droplet-based datasets) for clustering with BAMM-SC or competitors, and a reduced gene feature space must be defined before clustering.

## When NOT to use

- The goal is rare cell type identification — the article warns that filtering genes with excessive zeros 'may be problematic for rare cell type identification'
- The input is already a normalized or reduced feature matrix rather than raw droplet-based UMI counts, so re-filtering by zero fraction would double-filter
- The data are bulk RNA-seq or sequencing-depth-downsampled Cell Ranger aggregates, since downsampling in the aggregation step causes substantial information loss for individuals with higher sequencing depth

## Inputs

- Raw UMI count matrices per individual, stored as a list with each element a gene-by-cell count matrix (rows = genes, columns = cells), generated via the 10x Genomics Chromium system and Cell Ranger
- Cell-type-specific marker panels from Supplementary Tables 2-4 (optional, for marker-based ground-truth labeling)

## Outputs

- Filtered cell-by-gene count matrices restricted to the top 1000 highly variable genes, one shared gene set across all individuals per dataset
- Per-dataset marker-based type assignments and assignment rates (e.g., >70% coverage for PBMC, >66% for mouse lung, >67% for human skin) when the optional labeling step is run

## How to apply

Start from a list of per-individual gene-by-cell UMI count matrices (rows = genes, columns = cells; gene counts must match across individuals, cell counts may differ). First filter out genes with excessive zeros — i.e., genes expressed in fewer than 1% of cells — under the article's assumption that lowly-expressed genes do not contribute much to clustering. Then rank the remaining genes by variability and select the top 1000 highly variable genes as the clustering feature space, applying the identical procedure to each real dataset (human PBMC pooled from 5 donors, mouse lung, human skin). Keep the same gene set across all individuals in the list so BAMM-SC can be run as BAMMSC(X, K) on the shared feature space. Optionally, score cell-type-specific marker panels (Supplementary Tables 2-4) on this reduced matrix to assign approximated ground-truth labels, removing cells with unambiguous-type failures and recording per-dataset assignment rates.

## Related tools

- **BAMM-SC** (Downstream consumer of the top-1000-HVG count matrices; takes the list of per-individual UMI count matrices as X and clusters across individuals) — https://github.com/lichen-lab/BAMMSC
- **Cell Ranger** (Upstream 10x Genomics pipeline that produces the per-library UMI count matrices aggregated across individuals before filtering and HVG selection)

## Evaluation signals

- After selection, exactly 1000 genes remain per dataset, and every retained gene is detected in at least 1% of cells
- All individuals within a dataset share the identical 1000-gene set (same row order) in the list passed to BAMMSC()
- Marker-based labeling coverage on the reduced matrix exceeds the per-dataset thresholds: >70% for PBMC, >66% for mouse lung, >67% for human skin
- BAMM-SC run on the filtered matrices accepts the input without dimension errors and returns mem, AIC, and BIC outputs
- Downstream clustering on the HVG space yields improved clustering accuracy (adjustedRandIndex) relative to using unfiltered genes, as in the article's benchmarking

## Limitations

- Filtering out genes with excessive zeros may be problematic for rare cell type identification, per the article's discussion
- BAMM-SC does not explicitly model a zero-inflation pattern, which may or may not affect clustering accuracy on the retained genes
- The top-1000-HVG cutoff is a benchmarking convention from this paper, not a universal threshold; rare cell types may be underrepresented in the retained feature space

## Evidence

- [discussion] Rationale for zero filtering: "we filtered out genes with excessive zeros from the analysis under the assumption that lowly-expressed genes do not contribute much to clustering"
- [discussion] Caveat on zero filtering for rare types: "This may be problematic for rare cell type identification"
- [readme] Expected input format after HVG selection: "X: a list with each element a UMI count matrix for each individual, with row as number of genes and column as number of cells"
