---
name: single-cell-pca-dimensionality-reduction
description: Use when a normalized/scaled count matrix of barcoded single cells must
  be reduced to PC space before building a transcriptome nearest-neighbor graph for
  clustering or visualization (e.g., prior to ClonoCluster hybrid clustering or UMAP).
when_to_use_negative:
- Input is raw (not normalized or scaled) counts — the README specifies 'a normalized
  or scaled count matrix of your barcoded cells'
- Data are not single-cell profiles with cell-level barcodes, since the PC embedding
  is combined with clone-barcode weights downstream
- A full (non-truncated) PCA or biplot analysis is the goal; here irlba truncated
  SVD with npc = 25 is used only as a clustering preprocessing step
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: irlba_wrap (ClonoCluster wrapper)
  role: Computes the 25-component truncated PCA of the cell-by-gene matrix before
    clustering
  repo: https://github.com/leeprichman/ClonoCluster
- name: ClonoCluster
  role: Consumes the PCA output as transcriptome edge weights combined with clone
    weights at tunable alpha
  repo: https://github.com/leeprichman/ClonoCluster
- name: R
  role: Runtime environment (version >= 3.6) in which dt2m and irlba_wrap are executed
  repo: https://github.com/leeprichman/ClonoCluster
- name: devtools
  role: Used to install the ClonoCluster package containing dt2m and irlba_wrap helpers
  repo: https://github.com/leeprichman/ClonoCluster
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:5b95445c78131deff4e9e8cbc536f18e2f4545c711b18f913abfa9f6304be821
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leeprichman/ClonoCluster
  tools:
  - ClonoCluster
  - GapClust
  - R
  - R (version >=)
  - devtools
  - irlba
  - irlba_wrap (ClonoCluster wrapper)
  - magrittr
  - rflann
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# single-cell-pca-dimensionality-reduction

## Summary

Project a single-cell gene expression matrix into a low-dimensional space (25 principal components computed with irlba in the ClonoCluster workflow) so that downstream graph-based clustering operates on denoised cell embeddings rather than raw gene counts.

## When to use

Use when a normalized/scaled count matrix of barcoded single cells must be reduced to PC space before building a transcriptome nearest-neighbor graph for clustering or visualization (e.g., prior to ClonoCluster hybrid clustering or UMAP).

## When NOT to use

- Input is raw (not normalized or scaled) counts — the README specifies 'a normalized or scaled count matrix of your barcoded cells'
- Data are not single-cell profiles with cell-level barcodes, since the PC embedding is combined with clone-barcode weights downstream
- A full (non-truncated) PCA or biplot analysis is the goal; here irlba truncated SVD with npc = 25 is used only as a clustering preprocessing step

## Inputs

- Normalized or scaled count matrix (gene-by-cell or cell-by-gene text table, e.g., from Data_genes/ of the ClonoCluster_paper release)
- Table of unique cell IDs and their assigned barcodes (data.table-readable)

## Outputs

- 25-component principal component embedding (cells x 25 PC matrix) usable as transcriptome edge weights for clustering
- PC matrix compatible with engage_warp() for Warp Factor UMAP visualization

## How to apply

Read the normalized/scaled gene-by-cell count matrix with data.table::fread and convert it to a cell-by-gene matrix using the dt2m helper (cm %<>% dt2m). Compute 25 principal components via the irlba_wrap wrapper (pca <- irlba_wrap(cm, npc = 25)), which provides the approximate truncated SVD appropriate for sparse single-cell matrices. The resulting PC embedding serves as the transcriptome-based edge weights input: the transcriptome nearest-neighbor network graph is built from this PCA, and ClonoCluster later mixes these weights with barcode-derived clone weights using the tunable parameter alpha. The same PCA output also feeds the Warp Factor pipeline, where PC values are warped toward clone-cluster mean PCs for UMAP visualization.

## Related tools

- **irlba_wrap (ClonoCluster wrapper)** (Computes the 25-component truncated PCA of the cell-by-gene matrix before clustering) — https://github.com/leeprichman/ClonoCluster
- **ClonoCluster** (Consumes the PCA output as transcriptome edge weights combined with clone weights at tunable alpha) — https://github.com/leeprichman/ClonoCluster
- **R** (Runtime environment (version >= 3.6) in which dt2m and irlba_wrap are executed) — https://github.com/leeprichman/ClonoCluster
- **devtools** (Used to install the ClonoCluster package containing dt2m and irlba_wrap helpers) — https://github.com/leeprichman/ClonoCluster

## Examples

```
pca <- irlba_wrap(cm, npc = 25)
```

## Evaluation signals

- The PCA call uses exactly npc = 25 components as specified in the article's methods example
- The input matrix has been normalized or scaled (per README requirements) and correctly transposed to cell-by-gene via dt2m
- At alpha = 0, downstream ClonoCluster clustering of the PCA embedding matches transcriptome-only clustering at the chosen resolution
- The embedding contains all barcoded cells with IDs matching the barcode table, so clone weights can be joined 1:1 to PCA rows

## Limitations

- Only R version >= 3.6 is specified; version numbers for irlba and other helper functions are not stated
- The number of components (npc = 25) is a fixed choice from the paper's example; no adaptive component-selection procedure is described
- PCA here is a preprocessing step for network-graph edge weights; its utility for identifying systems with intrinsic determinants of cell fate was not reliably predictable in the study

## Evidence

- [methods] irlba_wrap pca on count matrix: "pca <- irlba_wrap(cm, npc = 25)"
- [readme] readme input requirements: "A normalized or scaled count matrix of your barcoded cells"
- [intro] role of PCA in the transcriptome graph: "the transcriptome nearest neighbor network graph edge weights are modified to incorporate clone clusters with a tunable free parameter, α"
