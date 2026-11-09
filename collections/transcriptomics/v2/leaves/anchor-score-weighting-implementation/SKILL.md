---
name: anchor-score-weighting-implementation
description: Use when reconstructing or validating how STACAS weights integration
  anchors during batch effect correction, or when you need to reproduce the sample
  guide tree that determines the order of multi-dataset Seurat integration.
when_to_use_negative:
- The task is purely unsupervised integration without any cell type annotations —
  the anchor weighting depends on label probability and label.confidence filtering
- You only need to evaluate integration quality post hoc (use CiLISI/ASW from carmonalab/scIntegrationMetrics
  instead of reconstructing anchor weighting)
- Datasets have only one sample pair where no guide tree ordering decision is needed
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: STACAS
  role: Source of FindAnchors.STACAS, FilterAnchors.STACAS, SampleTree.STACAS and
    Run.STACAS functions; provides sampleObj example data and the reference anchor
    weighting implementation
  repo: https://github.com/carmonalab/STACAS
- name: Seurat v4
  role: Integration framework in which STACAS anchors are computed and the guide tree
    drives Seurat-style integration
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:ad38b65f27ee5c20be48b7df756ea820389f80c3db4f3353bb6a9f974e3e3e4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/STACAS
  tools:
  - STACAS
  - Seurat v4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# anchor-score-weighting-implementation

## Summary

Implements STACAS's semi-supervised anchor weighting scheme for scRNA-seq integration: anchors are scored from rPCA and SNN similarities via a logistic transform (alpha = 0.8 geometric weighted log-sum), label-inconsistent anchors are rejected, and retained scores are summed into global dataset similarities that drive a ward.D2 guide tree fixing the integration order.

## When to use

Use when reconstructing or validating how STACAS weights integration anchors during batch effect correction, or when you need to reproduce the sample guide tree that determines the order of multi-dataset Seurat integration.

## When NOT to use

- The task is purely unsupervised integration without any cell type annotations — the anchor weighting depends on label probability and label.confidence filtering
- You only need to evaluate integration quality post hoc (use CiLISI/ASW from carmonalab/scIntegrationMetrics instead of reconstructing anchor weighting)
- Datasets have only one sample pair where no guide tree ordering decision is needed

## Inputs

- sampleObj example dataset shipped with the STACAS R package
- Seurat objects with cell type label annotations (prior knowledge) per dataset
- Pairwise integration anchors computed by FindAnchors.STACAS

## Outputs

- Weighted, label-confident anchor score set per dataset pair
- Global dataset-pair similarity matrix derived from summed anchor scores
- Rooted hierarchical guide tree (hclust ward.D2) determining integration order
- Comparison table (e.g. stacas_anchor_weighting_validation.csv) of independently computed tree/correction vectors vs Run.STACAS defaults

## How to apply

Load the sampleObj example dataset shipped with the STACAS package and compute pairwise integration anchors with FindAnchors.STACAS. Filter the anchors by pairwise rPCA distance using FilterAnchors.STACAS, since STACAS filters anchors based on this distance before weighting. Implement the weighting by transforming rPCA distance to a score via a logistic function, combining rPCA and SNN scores with a geometric weighted log-sum (alpha = 0.8), and rejecting anchors whose cell type label probability falls below the label.confidence threshold — this is the semi-supervised component that preserves biological variability. Build the guide tree by summing retained anchor scores over dataset pairs, clustering with hclust(method='ward.D2'), and rooting the tree at the dataset with the highest total anchor score, mirroring SampleTree.STACAS. Finally run Run.STACAS (the wrapper for the complete anchor-finding algorithm) on sampleObj and compare the independently implemented tree topology and correction vectors against the Run.STACAS defaults.

## Related tools

- **STACAS** (Source of FindAnchors.STACAS, FilterAnchors.STACAS, SampleTree.STACAS and Run.STACAS functions; provides sampleObj example data and the reference anchor weighting implementation) — https://github.com/carmonalab/STACAS
- **Seurat v4** (Integration framework in which STACAS anchors are computed and the guide tree drives Seurat-style integration)

## Examples

```
anchors <- FindAnchors.STACAS(sampleObj); anchors <- FilterAnchors.STACAS(anchors); tree <- SampleTree.STACAS(anchors); integrated <- Run.STACAS(sampleObj)
```

## Evaluation signals

- Independently reconstructed guide tree topology matches the sample.tree returned by Run.STACAS on sampleObj
- Anchor counts after FilterAnchors.STACAS match the pairwise rPCA distance filtering behavior of the reference implementation
- Anchors with cell type label probability below label.confidence are excluded from the summed scores
- Correction vectors from the independent implementation agree with Run.STACAS default output when compared in the validation table
- Tree is rooted at the dataset with the highest total (summed, weighted) anchor score

## Limitations

- Semi-supervised weighting requires cell type labels; anchors are rejected when label probability falls below label.confidence, so datasets lacking annotations lose the semi-supervised benefit
- CiLISI and similar cell type-aware evaluation metrics require cell type annotations and cannot be calculated for cell types present in only one dataset
- Batch mixing metrics robust to lopsided cell proportions are still lacking, complicating post-hoc assessment of the integration order

## Evidence

- [methods] FindAnchors.STACAS computes anchors between datasets: "`FindAnchors.STACAS`  Computes anchors between datasets for integration in Seurat"
- [methods] FilterAnchors.STACAS filters by rPCA distance: "`FilterAnchors.STACAS`  Filters integration anchors based on pairwise rPCA distance, calculated using `FindAnchors.STACAS`"
- [methods] SampleTree.STACAS determines the hierarchical integration tree: "`SampleTree.STACAS`   Determine automatically a hierarchical tree for Seurat integration using anchors calculated in STACAS"
- [intro] STACAS is a semi-supervised integration method leveraging cell type labels: "we describe STACAS, a semi-supervised scRNA-seq data integration method that leverages prior knowledge in the form of cell type annotations"
- [readme] README: STACAS corrects batch effects and leverages label knowledge: "STACAS is a method for scRNA-seq integration or batch effect correction."
- [abstract] Robustness to imperfect labels motivates label-confidence filtering: "STACAS scales well to large datasets and is robust to incomplete and imprecise input cell type labels, which are commonly encountered in real-life integration tasks."
