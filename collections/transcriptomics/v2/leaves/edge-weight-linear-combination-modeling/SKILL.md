---
name: edge-weight-linear-combination-modeling
description: Use when clustering barcoded single-cell data where a hybrid graph must
  weight both transcriptome nearest-neighbor similarity and shared clone-barcode relationships
  with a continuous α parameter.
when_to_use_negative:
- Cells carry no clone barcodes, so only the transcriptome jaccard graph is available
  and α cannot be varied meaningfully
- Data are full-lineage tracing (CRISPR-Cas9 genome editing or imaging-based family
  trees) without adaptation of clone weights to phylogenetic distance
- The goal is 2D visualization rather than clustering — Warp Factor modifying the
  PCA input to UMAP is the complementary tool for that task
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ClonoCluster
  role: R package providing clonocluster(), build_barcode_matrix, and dt2m to linearly
    combine transcriptome and clone edge weights with tunable α
  repo: https://github.com/leeprichman/ClonoCluster
- name: Seurat
  role: Builds the shared-nearest-neighbor jaccard-weighted transcriptome graph on
    the PCs
- name: irlba
  role: irlba_wrap computes the 25-PC representation of the count matrix feeding the
    SNN graph
- name: data.table
  role: fread loads the deposited barcode and count-matrix TSV files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:f621fdb2b1ccd3648e1c5decb59db8e8d736bf0de4b57a9951cce1326e34f3ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leeprichman/ClonoCluster
  tools:
  - ClonoCluster
  - R (version >=)
  - Seurat
  - data.table
  - irlba
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# edge-weight-linear-combination-modeling

## Summary

ClonoCluster's integration of transcriptome and clone barcode similarity into one weighted network graph by linearly combining Seurat SNN jaccard transcriptome edge weights with size-normalized clone weights via W = αβ(lin − J) + J, with tunable α and β = 0.1.

## When to use

Use when clustering barcoded single-cell data where a hybrid graph must weight both transcriptome nearest-neighbor similarity and shared clone-barcode relationships with a continuous α parameter.

## When NOT to use

- Cells carry no clone barcodes, so only the transcriptome jaccard graph is available and α cannot be varied meaningfully
- Data are full-lineage tracing (CRISPR-Cas9 genome editing or imaging-based family trees) without adaptation of clone weights to phylogenetic distance
- The goal is 2D visualization rather than clustering — Warp Factor modifying the PCA input to UMAP is the complementary tool for that task

## Inputs

- Two-column barcode table (columns 'rn' and 'Barcode') in TSV format
- Normalized/scaled gene-by-cell count matrix in TSV format (loadable with data.table::fread and convertible via ClonoCluster's dt2m)
- Tunable parameters α (relative transcriptome-vs-clone weight, swept from 0 to 1) and β (set to 0.1)

## Outputs

- Combined weighted adjacency matrix W = αβ(lin − J) + J for each α (e.g. clonocluster_weighted_adjacency.tsv)
- Hybrid cluster assignments across the α sweep

## How to apply

Compute a 25-principal-component representation of the cell-by-gene count matrix with irlba_wrap(cm, npc = 25), then build the Seurat shared-nearest-neighbor jaccard-weighted graph on the PCs to obtain transcriptome edge weights. Build the clone weight matrix with build_barcode_matrix, assigning each edge a clone weight of 1 or 0 for shared/different barcodes and normalizing by the number of cells carrying each barcode. Integrate with W = αβ(lin − J) + J using β = 0.1 and sweep α over seq(0, 1, by = 0.1) via clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1). Verify that at α = 0 the combined weights reproduce the transcriptome jaccard graph and at α = 1 they reproduce the size-normalized clone graph, then write the combined weighted adjacency matrix (e.g. clonocluster_weighted_adjacency.tsv).

## Related tools

- **ClonoCluster** (R package providing clonocluster(), build_barcode_matrix, and dt2m to linearly combine transcriptome and clone edge weights with tunable α) — https://github.com/leeprichman/ClonoCluster
- **Seurat** (Builds the shared-nearest-neighbor jaccard-weighted transcriptome graph on the PCs)
- **irlba** (irlba_wrap computes the 25-PC representation of the count matrix feeding the SNN graph)
- **data.table** (fread loads the deposited barcode and count-matrix TSV files)

## Examples

```
clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)
```

## Evaluation signals

- At α = 0 the combined matrix equals the transcriptome-only jaccard graph exactly
- At α = 1 the combined matrix equals the size-normalized clone-weight graph exactly
- All combined edge weights lie in [0, 1] for every α in seq(0, 1, by = 0.1)
- Plot of number of clusters returned vs increasing α behaves monotonically/sensibly, identifying the maximum tolerable α

## Limitations

- Whether the appropriate relative weighting of transcriptome vs clone information differs between endpoint and transitional systems remains unresolved and requires case-by-case evaluation
- The analysis did not address full-lineage tracing data (CRISPR-Cas9 genome editing or imaging-based family trees), though ClonoCluster could be adapted to weight distances by phylogenetic distance
- No reliable method was found to identify which experimental systems have intrinsic determinants of cell fate and would benefit from clonal information

## Evidence

- [intro] Linearly combine transcriptome and clone weights using α: "We linearly combined transcriptome and clone weights using α such that it returned purely transcriptome weights for α = 0 and purely clone weights for α = 1"
- [methods] clonocluster invocation with alpha sweep and beta = 0.1: "clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)"
- [intro] Clone weight definition and normalization: "For each edge between cells, we also created a “clone weight” of 1 or 0 based on whether the cells have the same or different barcodes. We then normalized the “clone weight” by the number of cells"
- [abstract] Tunable parameter weighting both data kinds: "a computational method that combines both clone and transcriptome information to create hybrid clusters that weight both kinds of data with a tunable parameter"
- [readme] Edge weight vs alpha figure script: [evidence span withheld — anchor status: external_doc]
