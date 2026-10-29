---
name: hierarchical-guide-tree-construction
description: Use when integrating multiple scRNA-seq samples with STACAS/Seurat and
  you need to reproduce or customize the automatic integration order derived from
  anchor scores instead of using an arbitrary or manual dataset order.
when_to_use_negative:
- No prior cell type labels are available and anchors cannot pass the label.confidence
  consistency filter, since the weighting rejects anchors with inconsistent cell type
  labels
- Datasets share few or no cell types, so pairwise anchors are too sparse to yield
  meaningful global similarities for tree construction
- Only two datasets are being integrated, where a guide tree ordering is unnecessary
edam_operation: http://edamontology.org/operation_0313
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STACAS
  role: Provides FindAnchors.STACAS, FilterAnchors.STACAS, SampleTree.STACAS and Run.STACAS
    for anchor scoring, filtering, and guide-tree determination of integration order
  repo: https://github.com/carmonalab/STACAS
- name: Seurat v4
  role: Underlying integration framework whose anchors and IntegrateData workflow
    the STACAS tree guides
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:22c3c0c45b34abef098b5e69b6d874781cb7b5d3ddd317ee4e1b53db2f91613a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/STACAS
  tools:
  - STACAS
  - Seurat v4
  edam_operation: http://edamontology.org/operation_0313
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# hierarchical-guide-tree-construction

## Summary

Reconstructs STACAS's guide-tree stage: pairwise integration anchors are weighted by rPCA distance and label consistency, summed into global dataset similarities, and clustered (ward.D2) to fix the order in which datasets are integrated.

## When to use

Use when integrating multiple scRNA-seq samples with STACAS/Seurat and you need to reproduce or customize the automatic integration order derived from anchor scores instead of using an arbitrary or manual dataset order.

## When NOT to use

- No prior cell type labels are available and anchors cannot pass the label.confidence consistency filter, since the weighting rejects anchors with inconsistent cell type labels
- Datasets share few or no cell types, so pairwise anchors are too sparse to yield meaningful global similarities for tree construction
- Only two datasets are being integrated, where a guide tree ordering is unnecessary

## Inputs

- Seurat object (or list of Seurat objects) per sample, e.g. the sampleObj example dataset shipped with STACAS
- Pairwise integration anchors from FindAnchors.STACAS with associated rPCA distances
- Cell type label annotations per cell (for label-confidence rejection of inconsistent anchors)
- Anchor score parameters: logistic transform of rPCA distance, SNN scores, alpha = 0.8, label.confidence cutoff

## Outputs

- Rooted hierarchical guide tree (hclust ward.D2 dendrogram) specifying dataset integration order, rooted at the dataset with the highest total anchor score
- Per-pair anchor score matrix (retained, weighted anchor scores summed over dataset pairs)
- Validated integration correction vectors from Run.STACAS compared against the independent implementation (e.g. stacas_anchor_weighting_validation.csv)

## How to apply

Compute pairwise anchors with FindAnchors.STACAS on Seurat objects (e.g. the sampleObj example data shipped with STACAS), then filter them by pairwise rPCA distance with FilterAnchors.STACAS. Weight each anchor by transforming its rPCA distance to a score via a logistic function, combine the rPCA and SNN scores with a geometric weighted log-sum using alpha = 0.8, and reject anchors whose label probability falls below label.confidence. Sum the retained, weighted anchor scores over each dataset pair to obtain global similarities between datasets, cluster these with hclust(method='ward.D2'), and root the resulting tree at the dataset with the highest total anchor score, mirroring SampleTree.STACAS. Validate the independently implemented tree topology and correction vectors against Run.STACAS defaults run on sampleObj, writing the comparison to stacas_anchor_weighting_validation.csv before using the tree to guide IntegrateData.STACAS.

## Related tools

- **STACAS** (Provides FindAnchors.STACAS, FilterAnchors.STACAS, SampleTree.STACAS and Run.STACAS for anchor scoring, filtering, and guide-tree determination of integration order) — https://github.com/carmonalab/STACAS
- **Seurat v4** (Underlying integration framework whose anchors and IntegrateData workflow the STACAS tree guides)

## Examples

```
anchors <- FindAnchors.STACAS(sampleObj); anchors <- FilterAnchors.STACAS(anchors); tree <- SampleTree.STACAS(anchors); integrated <- Run.STACAS(sampleObj)
```

## Evaluation signals

- The independently implemented tree topology matches the guide tree returned by Run.STACAS with default parameters on sampleObj
- The tree is rooted at the dataset with the highest total anchor score, matching SampleTree.STACAS behavior
- hclust(method='ward.D2') is used on the pair-summed weighted anchor scores, and anchors below label.confidence are excluded before summation
- Correction vectors obtained using the reconstructed tree match the Run.STACAS defaults, recorded in stacas_anchor_weighting_validation.csv
- Inspecting anchor distance distributions (PlotAnchors.STACAS) shows the FilterAnchors.STACAS rPCA-distance filtering retained plausible anchor sets for every dataset pair

## Limitations

- Guide-tree quality depends on label precision: STACAS is robust to incomplete and imprecise cell type labels, but inconsistent labels push anchors below label.confidence and can leave dataset pairs with no scored anchors
- Differences in cell type abundance across samples are a major, challenging component of biological variance and can weaken anchor-based global similarities used for ordering
- The tree determines only the integration order; downstream CiLISI-style evaluation requires cell type annotations and cannot be computed for cell types present in only one dataset

## Evidence

- [methods] SampleTree.STACAS determines a hierarchical tree for Seurat integration: "`SampleTree.STACAS`   Determine automatically a hierarchical tree for Seurat integration using anchors calculated in STACAS"
- [methods] FilterAnchors.STACAS filters anchors by pairwise rPCA distance: "`FilterAnchors.STACAS`  Filters integration anchors based on pairwise rPCA distance, calculated using `FindAnchors.STACAS`"
- [methods] FindAnchors.STACAS computes the pairwise anchors feeding the tree: "`FindAnchors.STACAS`  Computes anchors between datasets for integration in Seurat"
- [readme] Prior cell type labels support the semi-supervised weighting behind the tree: "Prior cell type knowledge, given as cell type labels, can be provided to the algorithm to perform semi-supervised integration, leading to increased preservation of biological variability"
- [intro] STACAS leverages cell type annotations to preserve biological variance: "we describe STACAS, a semi-supervised scRNA-seq data integration method that leverages prior knowledge in the form of cell type annotations"
