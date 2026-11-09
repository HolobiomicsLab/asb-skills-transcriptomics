---
name: unsupervised-feature-selection-for-scrnaseq
description: Use when clustering scRNA-seq expression matrices where all genes cannot
  be treated as equally significant and insignificant genes must be filtered out without
  using cell-type labels.
when_to_use_negative:
- Data is not raw gene-expression scRNA-seq (e.g. already a reduced feature embedding
  or a cell x gene annotation table) — the autoencoder and filter methods expect a
  gene expression matrix.
- Supervised or semi-supervised feature selection is desired — all four methods (Low
  Variance, Laplacian Score, SPEC, MCFS) are unsupervised and do not exploit labels.
- Only a single feature selection method can be afforded — the article's ablation
  showed single-method variants (e.g. SPEC alone) were second best and were outperformed
  by the four-method ensemble.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scEFSC
  role: Pipeline that orchestrates the non-negative kernel autoencoder pre-selection
    and the four unsupervised feature selections before ensemble clustering
  repo: https://github.com/Conan-Bian/scEFSC
- name: SC3
  role: One of nine base clustering algorithms consuming the feature-selected subsets
    within scEFSC
  repo: github.com/hemberg-lab/SC3
- name: CIDR
  role: Base clustering method (version 0.1.5) applied to feature-selected data
  repo: https://github.com/VCCRI/CIDR
- name: Seurat
  role: Base clustering method (v4.0.0, 10 reduced dimensions) applied to feature-selected
    data
- name: RaceID
  role: Base clustering method (v0.2.2, max 20 clusters) applied to feature-selected
    data
- name: wMetaC
  role: Weighted meta-clustering that combines base clustering labels built on the
    selected feature subsets
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_002
  - task_004
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.04.023
    title: 'scEFSC: Accurate single-cell RNA-seq data analysis via ensemble consensus
      clustering based on multiple feature selections'
  source_content_hash: sha256:4c95cac9a920ae83fce9361a153be2868abf61e350a9598ef34e1127f382ce67
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Conan-Bian/scEFSC
  repo_url: https://github.com/Conan-Bian/scEFSC
  tools:
  - CIDR
  - Monocle
  - PhenoGraph
  - RaceID
  - Rphenograph
  - SC3
  - SHARP
  - SINCERA
  - Seurat
  - pcaReduce
  - scDHA
  - scEFSC
  - wMetaC
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# unsupervised-feature-selection-for-scrnaseq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reduces scRNA-seq gene sets by first pre-selecting 5000 genes with a non-negative kernel autoencoder, then applying four unsupervised feature selection methods (Low Variance, Laplacian Score, SPEC, MCFS) to derive feature subsets for downstream ensemble clustering.

## When to use

Use when clustering scRNA-seq expression matrices where all genes cannot be treated as equally significant and insignificant genes must be filtered out without using cell-type labels.

## When NOT to use

- Data is not raw gene-expression scRNA-seq (e.g. already a reduced feature embedding or a cell x gene annotation table) — the autoencoder and filter methods expect a gene expression matrix.
- Supervised or semi-supervised feature selection is desired — all four methods (Low Variance, Laplacian Score, SPEC, MCFS) are unsupervised and do not exploit labels.
- Only a single feature selection method can be afforded — the article's ablation showed single-method variants (e.g. SPEC alone) were second best and were outperformed by the four-method ensemble.

## Inputs

- n × d scRNA-seq gene expression matrix (rows as genes, columns as cells/samples)
- Published cell-type labels for validation (optional, for ARI/NMI evaluation)

## Outputs

- Four feature subsets per dataset (one per selection method: Low Variance, Laplacian Score, SPEC, MCFS)
- Final per-cell cluster labels (e.g. scefsc_cluster_labels.csv) after ensemble clustering
- Ablation ARI/NMI table (ablation_ari_nmi_table.csv) comparing variants against full scEFSC

## How to apply

Load the n × d scRNA-seq expression matrix and preprocess it with a log2 transformation, removing genes detected in less than 2% of the cells (rescale with log2 if expression values exceed 100). Pre-select 5000 genes using a single-layer non-negative kernel autoencoder, retaining genes with large encoder-weight variances. Then apply four unsupervised feature selection algorithms — Low Variance (variance-based filter), Laplacian Score (Laplacian Eigenmaps/Locality Preserving Projection filter), SPEC (spectral graph theory framework), and MCFS (manifold learning with L1-regularization) — to derive four feature subsets per dataset. Feed each subset into multiple clustering algorithms and combine results via weighted meta-clustering (wMetaC, delta=0.01, ward.D). The rationale is that ensemble feature selection avoids depending on a single filtering criterion; ablations showed full scEFSC outperformed every single-feature-selection variant, with SPEC alone the second best.

## Related tools

- **scEFSC** (Pipeline that orchestrates the non-negative kernel autoencoder pre-selection and the four unsupervised feature selections before ensemble clustering) — https://github.com/Conan-Bian/scEFSC
- **SC3** (One of nine base clustering algorithms consuming the feature-selected subsets within scEFSC) — github.com/hemberg-lab/SC3
- **CIDR** (Base clustering method (version 0.1.5) applied to feature-selected data) — https://github.com/VCCRI/CIDR
- **Seurat** (Base clustering method (v4.0.0, 10 reduced dimensions) applied to feature-selected data)
- **RaceID** (Base clustering method (v0.2.2, max 20 clusters) applied to feature-selected data)
- **wMetaC** (Weighted meta-clustering that combines base clustering labels built on the selected feature subsets)

## Examples

```
Rscript run_scEFSC.R --input GSE81608_expression.csv --k 14 --fsmethods LowVariance,LaplacianScore,SPEC,MCFS --output scefsc_cluster_labels.csv
```

## Evaluation signals

- Confirm four distinct feature subsets are produced per dataset, one each from Low Variance, Laplacian Score, SPEC, and MCFS.
- Verify the autoencoder pre-selection retains exactly 5000 genes and that genes detected in <2% of cells were removed beforehand.
- Compare ARI/NMI of the ensemble output against the published labels; the full ensemble should achieve equal or better ARI/NMI than any single-feature-selection variant (e.g. best on six datasets under NMI, including GSE59739, GSE81608, GSE85241, GSE65525, GSE60361, phs000833.v3.p1).
- Check the ablation table (ablation_ari_nmi_table.csv) contains entries for 4 variants × 14 datasets × 2 metrics plus the full scEFSC ensemble.
- Ensure final labels show non-degenerate agreement (non-trivial NMI/ARI) with reference cell-type labels.

## Limitations

- Genes detected in less than 2% of cells must be removed after log2 normalization, or the selection operates on low-level expressed genes.
- Cells labeled ambiguously as 'abandoned' must be dropped, otherwise they contaminate the selection.
- Single feature selection methods alone underperform: full scEFSC outperformed all four single-method variants on six datasets under NMI and six under ARI.
- The number of cell types (k) must be known in advance for the downstream base clustering algorithms.

## Evidence
