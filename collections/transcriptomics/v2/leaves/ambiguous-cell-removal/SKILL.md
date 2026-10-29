---
name: ambiguous-cell-removal
description: Use when raw UMI count matrices from multiple individuals (e.g., a list
  of per-donor gene-by-cell count matrices) must be turned into marker-labeled approximated
  ground-truth cell identities for adjusted-Rand-index clustering benchmarks.
when_to_use_negative:
- When the goal is rare cell type identification — the authors note that filtering
  genes with excessive zeros 'may be problematic for rare cell type identification'
- When cells already have curated or authoritative ground-truth labels and marker-based
  approximated labels would only add noise
- When the input is already a filtered feature table rather than raw UMI count matrices
  from multiple individuals
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: BAMM-SC
  role: Downstream clustering method consuming the marker-labeled approximated ground-truth
    datasets for adjusted-Rand-index benchmarking
  repo: https://github.com/lichen-lab/BAMMSC
- name: 10x Genomics Chromium system
  role: Droplet-based platform that generated the in-house PBMC (and related tissue)
    scRNA-seq datasets being preprocessed
- name: Cell Ranger
  role: Pipeline used to aggregate the 10x libraries into the per-individual count
    matrices that serve as raw input
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:d8fdbb5c1c209968049b6a6d66763bd3e379d1239c350fa3bfb1a19c2a4b8ac3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/BAMMSC
  tools:
  - 10x Genomics Chromium system
  - BAMM-SC
  - Cell Ranger
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ambiguous-cell-removal

## Summary

Preprocesses real droplet-based scRNA-seq datasets (human PBMC, mouse lung, human skin) by filtering lowly-expressed genes, selecting the top 1000 HVGs, scoring cell-type marker panels, and removing cells whose marker signals do not support an unambiguous type so the surviving cells serve as approximated ground truth for clustering benchmarking.

## When to use

Use when raw UMI count matrices from multiple individuals (e.g., a list of per-donor gene-by-cell count matrices) must be turned into marker-labeled approximated ground-truth cell identities for adjusted-Rand-index clustering benchmarks.

## When NOT to use

- When the goal is rare cell type identification — the authors note that filtering genes with excessive zeros 'may be problematic for rare cell type identification'
- When cells already have curated or authoritative ground-truth labels and marker-based approximated labels would only add noise
- When the input is already a filtered feature table rather than raw UMI count matrices from multiple individuals

## Inputs

- Raw UMI count matrices per individual (list of gene-by-cell count matrices) for human PBMC, mouse lung, and human skin datasets
- Cell-type-specific marker panels from Supplementary Tables 2-4

## Outputs

- marker_labeled_datasets.rds containing labeled cell-by-gene matrices restricted to the top 1000 HVGs
- Per-dataset marker assignment rates for PBMC, mouse lung, and skin

## How to apply

Load the raw UMI count matrices for the three real datasets (human PBMC from 5 pooled donors, mouse lung, human skin), each stored as a list of per-individual gene-by-cell count matrices. For each dataset, filter out genes detected in fewer than 1% of cells — the article's rationale is that 'lowly-expressed genes do not contribute much to clustering' — then extract the top 1000 highly variable genes as the clustering feature space. Score the cell-type-specific marker panels (Supplementary Tables 2-4) against every cell to obtain marker-based type assignments. Remove cells whose marker signals do not support an unambiguous type, and record per-dataset marker assignment rates. Finally, save the labeled cell-by-gene matrices (top 1000 HVGs) and assignment rates to marker_labeled_datasets.rds; validation requires labeling coverage to exceed per-dataset thresholds (>70% for PBMC, >66% for mouse lung, >67% for human skin).

## Related tools

- **BAMM-SC** (Downstream clustering method consuming the marker-labeled approximated ground-truth datasets for adjusted-Rand-index benchmarking) — https://github.com/lichen-lab/BAMMSC
- **10x Genomics Chromium system** (Droplet-based platform that generated the in-house PBMC (and related tissue) scRNA-seq datasets being preprocessed)
- **Cell Ranger** (Pipeline used to aggregate the 10x libraries into the per-individual count matrices that serve as raw input)

## Examples

```
install_github("lichen-lab/BAMMSC"); library(BAMMSC); result=BAMMSC(data_BAMMSC,K=4)
```

## Evaluation signals

- Marker-based labeling coverage exceeds the per-dataset thresholds: >70% for PBMC, >66% for mouse lung, >67% for human skin
- Retained genes are detected in at least 1% of cells after zero-filtering
- Feature space is restricted to exactly the top 1000 highly variable genes per dataset
- Output file marker_labeled_datasets.rds parses as cell-by-gene matrices plus per-dataset assignment rates
- Downstream benchmarking clusters reproduce/compare against the marker labels via adjustedRandIndex

## Limitations

- Filtering genes with excessive zeros may be problematic for rare cell type identification
- Cells whose marker signals are ambiguous are discarded rather than assigned, so label coverage can vary across datasets
- BAMM-SC assumes each cell belongs to one distinct cluster; posterior probabilities cannot be interpreted as cell cycle or developmental stage quantification

## Evidence

- [discussion] filter rationale: "we filtered out genes with excessive zeros from the analysis under the assumption that lowly-expressed genes do not contribute much to clustering"
- [discussion] rare cell type caveat: "This may be problematic for rare cell type identification"
- [intro] dataset provenance: "We isolated from whole blood obtained from 4 healthy donors and used the 10x Chromium system to generate scRNA-seq data"
- [readme] clustering membership output structure: "mem: a list of vectors. Each vector [#cell] is the clustering membership for all cells in each individual. Length of list is #individuals"
- [methods] benchmark metric: "adjustedRandIndex(unlist(Zpred),unlist(Z))"
