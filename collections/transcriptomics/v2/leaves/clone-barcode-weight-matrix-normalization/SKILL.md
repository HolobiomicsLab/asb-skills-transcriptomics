---
name: clone-barcode-weight-matrix-normalization
description: Use when you have a two-column cell-ID-to-barcode table (e.g. columns
  'rn' and 'Barcode') and need to convert clonal relationships into normalized edge
  weights for graph-based single-cell clustering or integration with a transcriptome
  SNN graph.
when_to_use_negative:
- Cells have no clone barcode information (clustering must rely on transcriptome-only
  weights, α = 0)
- Input is full-lineage tracing data (CRISPR-Cas9 genome editing or imaging-based
  family trees) — the binary same/different barcode weight does not capture phylogenetic
  distance, though the authors note ClonoCluster could be adapted for this
- The barcode table is a gene-by-cell count matrix rather than a cell-ID-to-barcode
  assignment table
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: ClonoCluster
  role: Provides build_barcode_matrix to construct and size-normalize the clone weight
    matrix from barcode assignments
  repo: https://github.com/leeprichman/ClonoCluster
- name: data.table
  role: Reads the two-column barcode TSV (columns 'rn' and 'Barcode') into R
- name: R
  role: Runtime environment for the ClonoCluster package (version >= 3.6)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:e0f52306c75857f21ac6492204c8c2b2721888efbec603d479a7f32d235ec53b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leeprichman/ClonoCluster
  tools:
  - ClonoCluster
  - R
  - R (version >=)
  - Seurat
  - data.table
  - irlba
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# clone-barcode-weight-matrix-normalization

## Summary

Constructs the ClonoCluster 'clone weight' edge matrix for a cell-cell network graph: each edge receives a clone weight of 1 or 0 depending on whether two cells share the same barcode, then weights are normalized by the number of cells carrying that barcode, producing the clone modality input to the α-weighted hybrid graph.

## When to use

Use when you have a two-column cell-ID-to-barcode table (e.g. columns 'rn' and 'Barcode') and need to convert clonal relationships into normalized edge weights for graph-based single-cell clustering or integration with a transcriptome SNN graph.

## When NOT to use

- Cells have no clone barcode information (clustering must rely on transcriptome-only weights, α = 0)
- Input is full-lineage tracing data (CRISPR-Cas9 genome editing or imaging-based family trees) — the binary same/different barcode weight does not capture phylogenetic distance, though the authors note ClonoCluster could be adapted for this
- The barcode table is a gene-by-cell count matrix rather than a cell-ID-to-barcode assignment table

## Inputs

- Two-column barcode assignment table (cell ID column 'rn' and 'Barcode' column), e.g. TSV loaded with data.table::fread
- Cell-by-gene normalized/scaled count matrix (to define the cell universe matching the barcode table)

## Outputs

- Size-normalized clone weight matrix (cell-by-cell weighted adjacency, entries 0 or barcode-size-normalized 1/n values)
- Clone modality input to the ClonoCluster combined graph clonocluster(pca, bt, alpha, beta, res)

## How to apply

Load the barcode assignment table (two-column table of unique cell IDs and their assigned barcodes, e.g. with data.table::fread) and pass it to ClonoCluster's build_barcode_matrix to build the clone weight matrix. For each edge between cells, assign a clone weight of 1 when the cells share the same barcode and 0 when they have different barcodes, then normalize the clone weight by the number of cells carrying that barcode so that large clones do not dominate the graph. The resulting size-normalized clone weight matrix is combined with the transcriptome jaccard graph via W = αβ(lin − J) + J (with β = 0.1, α swept over seq(0, 1, by = 0.1)); at α = 1 the combined matrix must reproduce exactly the normalized clone-weight graph. Validate that all resulting edge weights lie in [0, 1].

## Related tools

- **ClonoCluster** (Provides build_barcode_matrix to construct and size-normalize the clone weight matrix from barcode assignments) — https://github.com/leeprichman/ClonoCluster
- **data.table** (Reads the two-column barcode TSV (columns 'rn' and 'Barcode') into R)
- **R** (Runtime environment for the ClonoCluster package (version >= 3.6))

## Examples

```
cm %<>% dt2m; pca <- irlba_wrap(cm, npc = 25); clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)
```

## Evaluation signals

- At α = 1 the combined weighted matrix equals the size-normalized clone-weight graph exactly
- Every edge weight is 0 for cells with different barcodes and equals 1/(number of cells with that barcode) for shared-barcode pairs after normalization
- All combined edge weights lie in [0, 1] for every α in seq(0, 1, by = 0.1)
- The clone weight matrix dimensions match the transcriptome SNN graph dimensions (cells in barcode table correspond one-to-one with count matrix columns)

## Limitations

- Normalization by clone size means very large clones contribute small per-edge weights; the study did not address phylogenetic or family-tree distance weighting
- The study did not reveal a reliable method to identify which experimental systems have intrinsic determinants of cell fate, so the appropriate relative weighting versus transcriptome information may require case-by-case evaluation (e.g. endpoint vs transitional systems)
- Twin analysis (barcoded twin cells in different local environments) is recommended as a critical control but was not part of the presented analyses

## Evidence

- [intro] For each edge between cells, we also created a “clone weight” of 1 or 0 based on whether the cells have the same or different barcodes. We then normalized the “clone weight” by the number of cells: "For each edge between cells, we also created a “clone weight” of 1 or 0 based on whether the cells have the same or different barcodes. We then normalized the “clone weight” by the number of cells"
- [intro] We linearly combined transcriptome and clone weights using α such that it returned purely transcriptome weights for α = 0 and purely clone weights for α = 1: "We linearly combined transcriptome and clone weights using α such that it returned purely transcriptome weights for α = 0 and purely clone weights for α = 1"
- [methods] clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1): "clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)"
- [readme] A table of unique cell IDs and their assigned barcodes: "Welcome to ClonoCluster. To get started, you will need:  1. A normalized or scaled count matrix of your barcoded cells  2. A table of unique cell IDs and their assigned barcodes"
- [discussion] Our analysis did not address these types of data, although it is relatively straightforward to adapt ClonoCluster to these situations: "Our analysis did not address these types of data, although it is relatively straightforward to adapt ClonoCluster to these situations"
