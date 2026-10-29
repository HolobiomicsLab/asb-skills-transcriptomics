---
name: single-cell-cluster-count-validation
description: Use when a published scRNA-seq result specifies an exact cell count and
  cluster number for a defined cell compartment (e.g., 44,799 T/NK cells in 32 clusters)
  and the goal is to independently reproduce and check those values in Seurat.
when_to_use_negative:
- The published result is not defined at the level of an annotated cell compartment
  (e.g., a whole-atlas clustering where subsetting is not part of the reported workflow)
- The task is cell-cell communication inference itself (LR-pair enrichment/visualization
  with scCrossTalk) rather than verifying the cell/cluster counts feeding into it
- The input is already a de novo clustering problem with no published reference cell/cluster
  counts to validate against
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Subsets the T/NK compartment from the integrated object and performs PCA,
    UMAP, and graph-based clustering for the cluster-count check; scCrossTalk is compatible
    with Seurat 5 objects
  repo: https://github.com/multitalk/scCrossTalk
- name: R
  role: Execution environment for the subsetting, clustering, and metric recording
    workflow (install scCrossTalk via devtools::install_github)
  repo: https://github.com/multitalk/scCrossTalk
- name: scCrossTalk
  role: Downstream consumer of the validated T/NK clustering — used afterwards to
    enrich and visualize LR pairs mediating cell-cell communications in the transplanted
    livers
  repo: https://github.com/multitalk/scCrossTalk
- name: devtools
  role: Installs scCrossTalk locally or from GitHub so the validated clusters can
    feed into communication inference
  repo: https://github.com/multitalk/scCrossTalk
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:c107362e68d8d335701df1446517f4d2004748b1967ac1fc617ea5fac9c4547e
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
  - devtools
  - scCrossTalk
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# single-cell-cluster-count-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A validation skill for reproducing reported single-cell subclustering results — subsetting an annotated cell compartment (here T/NK cells from 12 human liver transplant samples), re-running PCA/UMAP and graph-based clustering in Seurat, and confirming that cell counts (44,799 T/NK cells) and cluster numbers (32 distinctive clusters) match the published values before accepting the re-run.

## When to use

Use when a published scRNA-seq result specifies an exact cell count and cluster number for a defined cell compartment (e.g., 44,799 T/NK cells in 32 clusters) and the goal is to independently reproduce and check those values in Seurat.

## When NOT to use

- The published result is not defined at the level of an annotated cell compartment (e.g., a whole-atlas clustering where subsetting is not part of the reported workflow)
- The task is cell-cell communication inference itself (LR-pair enrichment/visualization with scCrossTalk) rather than verifying the cell/cluster counts feeding into it
- The input is already a de novo clustering problem with no published reference cell/cluster counts to validate against

## Inputs

- Integrated Seurat single-cell RNA-seq object for the 12 liver transplant samples (7 donors, cold perfusion and portal reperfusion)
- Annotated cell-type labels identifying the T/NK compartment (including MAIT cells and GZMB+ GZMK+ NK cells)
- Reported reference values: 44,799 T/NK cells and 32 distinctive clusters

## Outputs

- tnk_subcluster_metrics.csv recording the reproduced T/NK cell count and number of distinctive clusters
- Re-clustered Seurat subset object with PCA/UMAP embeddings and graph-based cluster assignments

## How to apply

Load the integrated Seurat single-cell RNA-seq object (12 liver samples from 7 donors at cold perfusion and portal reperfusion stages) and subset the target compartment — the T/NK compartment including MAIT cells and GZMB+ GZMK+ NK cells — using the annotated cell-type labels. Re-run dimensionality reduction (PCA followed by UMAP) and graph-based clustering on the subsetted object with Seurat, keeping parameters consistent with the publication. Count the total cells and the number of distinctive clusters recovered, then compare against the reported values of 44,799 cells and 32 clusters. Accept the run only when both values match, and record both in tnk_subcluster_metrics.csv so downstream cell-cell communication analyses (e.g., scCrossTalk LR-pair inference between hepatocytes, endothelial cells, MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils) operate on the verified compartment.

## Related tools

- **Seurat** (Subsets the T/NK compartment from the integrated object and performs PCA, UMAP, and graph-based clustering for the cluster-count check; scCrossTalk is compatible with Seurat 5 objects) — https://github.com/multitalk/scCrossTalk
- **R** (Execution environment for the subsetting, clustering, and metric recording workflow (install scCrossTalk via devtools::install_github)) — https://github.com/multitalk/scCrossTalk
- **scCrossTalk** (Downstream consumer of the validated T/NK clustering — used afterwards to enrich and visualize LR pairs mediating cell-cell communications in the transplanted livers) — https://github.com/multitalk/scCrossTalk
- **devtools** (Installs scCrossTalk locally or from GitHub so the validated clusters can feed into communication inference) — https://github.com/multitalk/scCrossTalk

## Examples

```
tnk <- subset(integrated_obj, idents = "T/NK"); tnk <- RunPCA(tnk); tnk <- RunUMAP(tnk, dims = 1:30); tnk <- FindClusters(tnk); write.csv(data.frame(cells = ncol(tnk), clusters = length(unique(Idents(tnk)))), "tnk_subcluster_metrics.csv")
```

## Evaluation signals

- Re-run cell count equals exactly 44,799 T/NK cells after subsetting the 12-sample integrated object
- Number of distinctive clusters from the re-run graph-based clustering equals 32
- Both reproduced values are recorded in tnk_subcluster_metrics.csv before acceptance
- The subset composition retains MAIT cells and GZMB+ GZMK+ NK cells per the annotated labels in the Seurat object
- PCA/UMAP embeddings are recomputed on the subset (not inherited unchanged from the full 75,231-cell atlas) before clustering

## Limitations

- Cluster counts from graph-based Seurat clustering depend on resolution and integration choices, so exact reproduction requires matching the original dimensionality-reduction and clustering settings
- No changelog exists for the source repository (missing artifact), so tool-version drift between Seurat versions cannot be tracked from the repo alone; note scCrossTalk states compatibility with Seurat 5
- The validation confirms counts only — it does not verify cluster identities or the biological interpretation of the EAD-associated immune niche

## Evidence

- [other] Re-run dimensionality reduction (PCA followed by UMAP) and graph-based clustering on the subsetted T/NK cells with Seurat.: [evidence span withheld — anchor status: not_found]
- [intro] Intro statement of scCrossTalk purpose: [evidence span withheld — anchor status: not_found]
