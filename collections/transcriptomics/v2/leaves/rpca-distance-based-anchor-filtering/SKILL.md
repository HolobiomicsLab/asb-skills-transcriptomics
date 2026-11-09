---
name: rpca-distance-based-anchor-filtering
description: Use when integrating multiple single-cell RNA-seq samples with STACAS/Seurat
  and you must remove anchors whose pairwise rPCA distance or inconsistent cell type
  labels make them unreliable before scoring anchors and building the integration
  guide tree.
when_to_use_negative:
- Input is a single dataset with no batch structure to correct, so there are no cross-sample
  anchors to filter
- The integration method does not use anchor-based correspondence (e.g., scVI/scANVI
  latent-space correction), where rPCA anchor filtering has no analogue
- Cell type annotations are entirely absent and unsupervised integration is desired
  — while STACAS is described as robust to missing labels, semi-supervised anchor
  confidence filtering relies on prior cell type information
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STACAS
  role: Provides FindAnchors.STACAS, FilterAnchors.STACAS, PlotAnchors.STACAS, SampleTree.STACAS,
    and Run.STACAS for anchor computation, rPCA-distance filtering, and guide-tree-based
    integration
  repo: https://github.com/carmonalab/STACAS
- name: Seurat v4
  role: Underlying framework in which STACAS anchors are computed between datasets
    and integration is performed (IntegrateData.STACAS targets Seurat integration)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:4ca63ab603283122eedaaee8add4d008652aae99c7905adfe4abe90c5835d73a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/STACAS
  tools:
  - STACAS
  - Seurat v4
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# rPCA distance-based anchor filtering (STACAS)

## Summary

Filters integration anchors between scRNA-seq datasets by pairwise rPCA distance using STACAS's FilterAnchors.STACAS, retaining only reliable cross-sample cell correspondences that then drive anchor scoring and guide-tree-based integration order.

## When to use

Use when integrating multiple single-cell RNA-seq samples with STACAS/Seurat and you must remove anchors whose pairwise rPCA distance or inconsistent cell type labels make them unreliable before scoring anchors and building the integration guide tree.

## When NOT to use

- Input is a single dataset with no batch structure to correct, so there are no cross-sample anchors to filter
- The integration method does not use anchor-based correspondence (e.g., scVI/scANVI latent-space correction), where rPCA anchor filtering has no analogue
- Cell type annotations are entirely absent and unsupervised integration is desired — while STACAS is described as robust to missing labels, semi-supervised anchor confidence filtering relies on prior cell type information

## Inputs

- Seurat sample objects (e.g., the sampleObj example dataset shipped with STACAS)
- Anchor set computed by FindAnchors.STACAS
- Cell type labels used for semi-supervised anchor confidence assessment

## Outputs

- Filtered anchor set (pairwise rPCA-distance-filtered anchors per dataset pair)
- Anchor distance distribution plots (PlotAnchors.STACAS)
- Weighted anchor scores feeding the integration guide tree and Run.STACAS correction vectors

## How to apply

Compute pairwise anchors between Seurat sample objects with FindAnchors.STACAS, then pass the anchor set to FilterAnchors.STACAS, which discards anchors based on their pairwise rPCA distance. Inspect anchor distance distributions with PlotAnchors.STACAS to sanity-check the filtering. For downstream weighting, transform the rPCA distance to a score via a logistic function and combine it with the SNN score using a geometric weighted log-sum with alpha = 0.8, rejecting anchors whose label probability falls below label.confidence. The sum of retained, weighted anchor scores yields global inter-dataset similarities used by SampleTree.STACAS, which clusters datasets with hclust(method='ward.D2') to determine integration order for Run.STACAS/IntegrateData.STACAS.

## Related tools

- **STACAS** (Provides FindAnchors.STACAS, FilterAnchors.STACAS, PlotAnchors.STACAS, SampleTree.STACAS, and Run.STACAS for anchor computation, rPCA-distance filtering, and guide-tree-based integration) — https://github.com/carmonalab/STACAS
- **Seurat v4** (Underlying framework in which STACAS anchors are computed between datasets and integration is performed (IntegrateData.STACAS targets Seurat integration))

## Examples

```
anchors <- FindAnchors.STACAS(sampleObj); filtered.anchors <- FilterAnchors.STACAS(anchors)
```

## Evaluation signals

- Filtered anchors match the output of FilterAnchors.STACAS applied to the FindAnchors.STACAS anchor set for the same pairwise rPCA-distance threshold
- PlotAnchors.STACAS distance distributions show that high-distance anchors were removed and low-distance anchors retained per dataset pair
- Independently built guide tree (summed weighted anchor scores, hclust(method='ward.D2'), rooted at highest total anchor score) matches the tree returned by Run.STACAS defaults
- Integration correction vectors from the reimplementation agree with Run.STACAS defaults, tabulated in stacas_anchor_weighting_validation.csv
- Anchors with inconsistent cell type labels (below label.confidence) are rejected from the weighted score

## Limitations

- Anchor filtering and weighting rely on label.confidence, so anchors between datasets with incomplete or imprecise cell type labels may be lost — although STACAS is documented as robust to such labels
- rPCA-distance filtering only assesses anchor reliability at the pair level; it does not itself resolve differences in cell type abundance across samples, a challenging component of biological variance
- The derived guide tree determines integration order but filtering/scores depend on Seurat anchor computation quality and the chosen dims and alpha = 0.8 weighting

## Evidence

- [methods] FilterAnchors filters by rPCA distance: "`FilterAnchors.STACAS`  Filters integration anchors based on pairwise rPCA distance, calculated using `FindAnchors.STACAS`"
- [methods] SampleTree determines integration tree from STACAS anchors: "`SampleTree.STACAS`   Determine automatically a hierarchical tree for Seurat integration using anchors calculated in STACAS"
- [readme] STACAS is robust to incomplete labels: "STACAS scales well to large datasets and is robust to incomplete and imprecise input cell type labels, which are commonly encountered in real-life integration tasks."
- [abstract] STACAS leverages prior cell type knowledge: "In this work we present STACAS, a batch correction method for scRNA-seq that leverages prior knowledge on cell types to preserve biological variability upon integration."
