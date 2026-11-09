---
name: single-cell-dimensionality-reduction
description: Use when an annotated single-cell Seurat object must be subsetted to
  a compartment (e.g., T/NK cells from the 12 liver transplant samples) and re-embedded/re-clustered
  via PCA -> UMAP and graph-based clustering to reproduce reported cluster counts.
when_to_use_negative:
- The goal is ligand-receptor cell-cell communication inference (scCrossTalk find_lrpairs
  / get_top_lrpairs / rev_gene), not re-clustering an annotated compartment.
- The input lacks annotated cell-type labels, since subsetting the T/NK compartment
  (including MAIT cells and GZMB+ GZMK+ NK cells) depends on prior Seurat annotations.
- The input is not a per-cell single-cell object (e.g., bulk RNA-seq or an already-reduced
  feature table).
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Subsets the T/NK compartment via annotated cell-type labels and performs the
    PCA, UMAP, and graph-based clustering re-run
- name: R
  role: Execution environment for the Seurat re-clustering and for writing tnk_subcluster_metrics.csv
- name: scCrossTalk
  role: Toolchain behind the source liver-transplant single-cell landscape from which
    the integrated object and annotations derive; not invoked in the PCA/UMAP re-run
    itself
  repo: https://github.com/multitalk/scCrossTalk
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:a76c46e30e05e3fa29f0bea2505e8f2b724aa24b19f6b3ea1b03a6a57e06206b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/multitalk/scCrossTalk
  repo_url: https://github.com/multitalk/scCrossTalk
  tools:
  - R
  - Seurat
  - scCrossTalk
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-dimensionality-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Re-runs Seurat-based dimensionality reduction (PCA followed by UMAP) and graph-based clustering on the T/NK compartment (including MAIT cells and GZMB+ GZMK+ NK cells) subsetted from the integrated 12-sample, 7-donor liver transplant scRNA-seq landscape, expecting 44,799 T/NK cells partitioned into 32 distinctive clusters.

## When to use

Use when an annotated single-cell Seurat object must be subsetted to a compartment (e.g., T/NK cells from the 12 liver transplant samples) and re-embedded/re-clustered via PCA -> UMAP and graph-based clustering to reproduce reported cluster counts.

## When NOT to use

- The goal is ligand-receptor cell-cell communication inference (scCrossTalk find_lrpairs / get_top_lrpairs / rev_gene), not re-clustering an annotated compartment.
- The input lacks annotated cell-type labels, since subsetting the T/NK compartment (including MAIT cells and GZMB+ GZMK+ NK cells) depends on prior Seurat annotations.
- The input is not a per-cell single-cell object (e.g., bulk RNA-seq or an already-reduced feature table).

## Inputs

- Integrated single-cell RNA-seq object (Seurat) for the 12 liver samples from 7 donors, collected during cold perfusion and portal reperfusion stages
- Annotated cell-type labels identifying the T/NK compartment, including MAIT cells and GZMB+ GZMK+ NK cells

## Outputs

- tnk_subcluster_metrics.csv recording the total T/NK cell count and the number of distinctive clusters
- PCA/UMAP embeddings and graph-based cluster assignments for the T/NK subset (44,799 cells, 32 clusters expected)

## How to apply

Load the integrated single-cell RNA-seq object covering the 12 liver samples from 7 donors (collected during cold perfusion and portal reperfusion stages) in Seurat, then subset the T/NK compartment using the annotated cell-type labels, taking care to retain MAIT cells and GZMB+ GZMK+ NK cells. Re-run dimensionality reduction - PCA followed by UMAP - and graph-based clustering (e.g., Seurat FindNeighbors/FindClusters) on the subsetted T/NK cells. Count the total number of T/NK cells and the number of distinctive clusters; the paper's reference run recovers 44,799 T/NK cells in 32 clusters. Write both values to tnk_subcluster_metrics.csv and accept the run only when both match exactly. Note that the exact number of PCs and clustering resolution are not specified in the provided context, and the README documents compatibility with Seurat 5.

## Related tools

- **Seurat** (Subsets the T/NK compartment via annotated cell-type labels and performs the PCA, UMAP, and graph-based clustering re-run)
- **R** (Execution environment for the Seurat re-clustering and for writing tnk_subcluster_metrics.csv)
- **scCrossTalk** (Toolchain behind the source liver-transplant single-cell landscape from which the integrated object and annotations derive; not invoked in the PCA/UMAP re-run itself) — https://github.com/multitalk/scCrossTalk

## Examples

```
library(Seurat); tnk <- subset(liver_obj, celltype %in% tnk_labels); tnk <- RunPCA(tnk); tnk <- RunUMAP(tnk); tnk <- FindNeighbors(tnk); tnk <- FindClusters(tnk)
```

## Evaluation signals

- Re-run cell count after subsetting the T/NK compartment equals 44,799 cells.
- Number of distinctive clusters recovered by the re-run equals 32.
- tnk_subcluster_metrics.csv exists and records both the cell count and cluster count values.
- The subset retains MAIT cells and GZMB+ GZMK+ NK cells, i.e., the labels used for subsetting cover the EAD-associated niche populations.
- UMAP is computed on the PCA reduction (PCA before UMAP), matching the workflow's stated ordering.

## Limitations

- Exact-match acceptance (44,799 cells; 32 clusters) is brittle: upstream integration or annotation differences can shift the counts.
- The number of PCs and the clustering resolution are not stated in the provided context, so the distinctive-cluster count may vary without the original Seurat parameters.
- The reference values are specific to the 12-sample, 7-donor liver transplant dataset (cold perfusion and portal reperfusion stages) and do not generalize to other tissues or cohorts.

## Evidence

- [other] Re-run dimensionality reduction (PCA followed by UMAP) and graph-based clustering on the subsetted T/NK cells with Seurat.: [evidence span withheld — anchor status: not_found]
- [other] Filtering and integrating T/NK cells from the 12 liver samples yielded 44,799 T/NK cells that were partitioned into 32 distinctive clusters by dimensionality reduction and clustering analysis.: [evidence span withheld — anchor status: not_found]
