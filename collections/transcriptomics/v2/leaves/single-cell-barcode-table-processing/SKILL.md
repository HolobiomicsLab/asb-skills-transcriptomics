---
name: single-cell-barcode-table-processing
description: Use when you have barcoded single-cell profiles with a table of unique
  cell IDs and assigned barcodes, and you want clonal relationships to inform clustering
  (alpha) or UMAP embeddings (Warp Factor s in 0-10).
when_to_use_negative:
- Cells lack lineage barcode assignments (no 'rn'/'Barcode' table), so only transcriptome-only
  clustering is possible
- Input is full-lineage tracing data (CRISPR-Cas9 genome editing or imaging-based
  family trees) rather than single-cell barcodes; the paper did not address these,
  though ClonoCluster could be adapted to weight by phylogenetic distance
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ClonoCluster
  role: R package that consumes the barcode table and PCA matrix to produce hybrid
    clusters weighted by the tunable parameter alpha; includes engage_warp for warp-factor
    UMAP
  repo: https://github.com/leeprichman/ClonoCluster
- name: Warp Factor
  role: Complementary tool that warps each cell's PCs toward its clone-cluster mean
    PC value (Ms = M - (s/10)(M - Mb)) prior to UMAP
  repo: https://github.com/leeprichman/ClonoCluster
- name: UMAP
  role: 2D visualization run on the warped PCA matrices via uwot/umap_matrix wrapped
    by engage_warp
- name: data.table
  role: CRAN package used with fread to load the count matrix and barcode table and
    rbindlist to combine per-warp embeddings
- name: Docker
  role: Prebuilt docker image (leeprichman/clonocluster) alternative for running the
    package without local installation
  repo: https://github.com/leeprichman/ClonoCluster
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:d0c631e2688663f9c6c2d8667ac5bc276927f84258cc3a75dbe67b7d25d16a60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leeprichman/ClonoCluster
  tools:
  - ClonoCluster
  - Docker
  - R (version >=)
  - UMAP
  - Warp Factor
  - data.table
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# single-cell-barcode-table-processing

## Summary

Prepares the two-column single-cell barcode assignment table (cell ID 'rn' and 'Barcode') and the paired PCA matrix (25 PCs via irlba_wrap) so ClonoCluster and Warp Factor can incorporate clonal origin into transcriptome clustering and UMAP visualization via the tunable parameters alpha and s.

## When to use

Use when you have barcoded single-cell profiles with a table of unique cell IDs and assigned barcodes, and you want clonal relationships to inform clustering (alpha) or UMAP embeddings (Warp Factor s in 0-10).

## When NOT to use

- Cells lack lineage barcode assignments (no 'rn'/'Barcode' table), so only transcriptome-only clustering is possible
- Input is full-lineage tracing data (CRISPR-Cas9 genome editing or imaging-based family trees) rather than single-cell barcodes; the paper did not address these, though ClonoCluster could be adapted to weight by phylogenetic distance

## Inputs

- Normalized or scaled count matrix of barcoded cells (gene x cell, retrievable from a Seurat object)
- Two-column table of unique cell IDs ('rn') and assigned barcodes ('Barcode')
- PCA matrix (25 PCs from irlba_wrap on the dt2m-converted count matrix)

## Outputs

- Hybrid cluster assignments per cell across a range of alpha values (e.g. *_cluster_assignments.txt)
- Warped UMAP 2D embeddings per warp factor (e.g. umap_warped_embeddings.tsv with a 'warp' column)
- Cluster marker AUCs and reorganization markers (e.g. *_auc_linclust.txt, *_auc_barcodes.txt)

## How to apply

Load the barcode table (two columns: 'rn' and 'Barcode') and the count matrix with data.table::fread in R (>= 3.6), converting the count matrix via dt2m and running PCA with irlba_wrap at npc = 25. Pass the PCA matrix and barcode table (bt) to clonocluster with alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1 to produce hybrid clusters that linearly combine transcriptome and clone edge weights (alpha = 0 purely transcriptome, alpha = 1 purely clone). For visualization, run engage_warp on the same inputs for warp factors s in {0, 5, 10}, which shifts each cell's PCs toward the mean PC of its clone cluster by fraction s/10 before UMAP. The rationale is that shared barcodes supply clonal origin, allowing the algorithm to reveal 'hidden' genes associated with cell fate; validation uses Plot_alluvia, cast_confusion, and Find_Markers_ROC.

## Related tools

- **ClonoCluster** (R package that consumes the barcode table and PCA matrix to produce hybrid clusters weighted by the tunable parameter alpha; includes engage_warp for warp-factor UMAP) — https://github.com/leeprichman/ClonoCluster
- **Warp Factor** (Complementary tool that warps each cell's PCs toward its clone-cluster mean PC value (Ms = M - (s/10)(M - Mb)) prior to UMAP) — https://github.com/leeprichman/ClonoCluster
- **UMAP** (2D visualization run on the warped PCA matrices via uwot/umap_matrix wrapped by engage_warp)
- **data.table** (CRAN package used with fread to load the count matrix and barcode table and rbindlist to combine per-warp embeddings)
- **Docker** (Prebuilt docker image (leeprichman/clonocluster) alternative for running the package without local installation) — https://github.com/leeprichman/ClonoCluster

## Examples

```
pca <- irlba_wrap(cm, npc = 25); umap_wf0 <- engage_warp(pca, bt, 0)
```

## Evaluation signals

- The warp factor s = 0 embedding reproduces the default UMAP of the unmodified 25-PC matrix
- Cells sharing the same barcode visibly/numerically coalesce in the UMAP as s increases toward 10, where clone-cluster variation is the only variation
- Barcode-to-cluster concordance (cast_confusion) and cohen's kappa vs alpha plots confirm increasing clonal weighting
- Marker fidelity checks (Find_Markers_ROC, Plot_alluvia_track Sankeys) show hybrid-cluster markers have equivalent fidelity to transcriptome-only markers
- Plotting number of clusters vs alpha identifies the maximum tolerable alpha before clusters fragment

## Limitations

- The appropriate relative weighting between transcriptome and clone information may differ between endpoint and transitional systems and requires case-by-case evaluation
- No reliable method was found to identify which experimental systems have intrinsic determinants of cell fate and would benefit from clonal information
- Software version numbers other than R (>= 3.6) are not stated; 'twin' barcoded-cell controls are recommended but were not part of the presented analyses

## Evidence

- [readme] Input barcode table requirement: "2. A table of unique cell IDs and their assigned barcodes"
- [methods] PCA construction for warping: "pca <- irlba_wrap(cm, npc = 25)"
- [methods] Hybrid clustering invocation with barcode table: "clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)"
- [methods] Warp-factor UMAP invocation: "umap_wf0 <- engage_warp(pca, bt, 0)"
- [intro] Warp Factor mechanism description: "a tunable “Warp Factor” parameter, s, warps the values of PCs for each cell to approach the mean value of the principal component for the clone cluster it belongs to"
