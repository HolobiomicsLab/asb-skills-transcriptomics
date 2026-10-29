---
name: t-nk-cell-subsetting-from-integrated-object
description: Use when an annotated integrated Seurat object from human transplanted-liver
  samples must be subsetted to its T/NK compartment (T cells, NK cells, MAIT, GZMB+
  GZMK+ NK) and re-clustered to resolve fine-grained subpopulations.
when_to_use_negative:
- The input is raw per-sample expression matrices that have not yet been integrated
  across the 12 liver samples — integrate first before subsetting annotated compartments.
- The analysis targets other compartments (e.g. hepatocytes, endothelial cells, S100A12+
  neutrophils) rather than the T/NK compartment with MAIT and GZMB+ GZMK+ NK cells.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Subsets the T/NK compartment via annotated cell-type labels and re-runs PCA,
    UMAP, and graph-based clustering on the subsetted cells
  repo: https://github.com/multitalk/scCrossTalk
- name: R
  role: Runtime environment in which the Seurat subsetting, dimensionality reduction,
    and clustering workflow is executed
  repo: https://github.com/multitalk/scCrossTalk
- name: scCrossTalk
  role: Downstream cell-cell communication inference applied to the clustered subpopulations;
    must be compatible with the Seurat version used for subclustering
  repo: https://github.com/multitalk/scCrossTalk
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:709a11da2178048c4c9e330ac4e199bd5ed64ddf4920279f73166d84bb5397e7
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# t-nk-cell-subsetting-from-integrated-object

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Subsets the T/NK cell compartment (including MAIT cells and GZMB+ GZMK+ NK cells) from a Seurat-integrated scRNA-seq object built from 12 liver samples collected from 7 donors during cold perfusion and portal reperfusion, then re-runs PCA, UMAP, and graph-based clustering to recover 32 distinctive clusters from 44,799 T/NK cells.

## When to use

Use when an annotated integrated Seurat object from human transplanted-liver samples must be subsetted to its T/NK compartment (T cells, NK cells, MAIT, GZMB+ GZMK+ NK) and re-clustered to resolve fine-grained subpopulations.

## When NOT to use

- The input is raw per-sample expression matrices that have not yet been integrated across the 12 liver samples — integrate first before subsetting annotated compartments.
- The analysis targets other compartments (e.g. hepatocytes, endothelial cells, S100A12+ neutrophils) rather than the T/NK compartment with MAIT and GZMB+ GZMK+ NK cells.

## Inputs

- Integrated single-cell RNA-seq object (Seurat) covering 12 liver samples from 7 donors (cold perfusion and portal reperfusion stages)
- Annotated cell-type labels embedded in the Seurat object (identifying T cells, NK cells, MAIT cells, GZMB+ GZMK+ NK cells)

## Outputs

- Subsetted Seurat object containing 44,799 T/NK cells
- 32 distinctive T/NK subclusters from PCA + UMAP + graph-based clustering
- tnk_subcluster_metrics.csv recording the cell count (44,799) and cluster count (32)

## How to apply

Load the Seurat-integrated scRNA-seq object for the 12 liver samples and subset the T/NK compartment using the annotated cell-type labels already stored in the object, explicitly retaining MAIT cells and GZMB+ GZMK+ NK cells. On the subsetted object, re-run dimensionality reduction (PCA followed by UMAP) and graph-based clustering in Seurat (scCrossTalk compatibility notes it works with Seurat 5). Count total T/NK cells and the number of distinctive clusters recovered, and record both in tnk_subcluster_metrics.csv. The run is accepted only when the cell count equals 44,799 and the cluster count equals 32; any deviation indicates incorrect subsetting or clustering parameters and must be resolved before proceeding.

## Related tools

- **Seurat** (Subsets the T/NK compartment via annotated cell-type labels and re-runs PCA, UMAP, and graph-based clustering on the subsetted cells) — https://github.com/multitalk/scCrossTalk
- **R** (Runtime environment in which the Seurat subsetting, dimensionality reduction, and clustering workflow is executed) — https://github.com/multitalk/scCrossTalk
- **scCrossTalk** (Downstream cell-cell communication inference applied to the clustered subpopulations; must be compatible with the Seurat version used for subclustering) — https://github.com/multitalk/scCrossTalk

## Examples

```
tnk <- subset(liver_integrated, cells = WhichCells(liver_integrated, expression = celltype %in% c("T", "NK", "MAIT", "GZMB+ GZMK+ NK"))); tnk <- FindClusters(tnk)
```

## Evaluation signals

- Re-run cell count equals 44,799 T/NK cells exactly.
- Number of distinctive clusters recovered equals 32.
- Subset retains MAIT cells and GZMB+ GZMK+ NK cells from the annotated labels, not just canonical T and NK cells.
- PCA and UMAP were re-run on the subset (not inherited from the full-object embeddings).
- Both values are written to tnk_subcluster_metrics.csv for validation.

## Limitations

- The 44,799-cell / 32-cluster result is specific to the 12 liver samples from 7 donors in this liver-transplantation cohort; other datasets will yield different counts and cannot use these as fixed acceptance values.
- Cluster number depends on Seurat graph-based clustering parameters; deviating resolutions will not reproduce 32 clusters.
- Downstream tools require Seurat 5 compatibility (noted for scCrossTalk), so version mismatches may break the workflow.

## Evidence

- [results] Subsetting yields 44,799 cells and 32 clusters: [evidence span withheld — anchor status: not_found]
- [other] Subset must include MAIT and GZMB+ GZMK+ NK cells: [evidence span withheld — anchor status: not_found]
