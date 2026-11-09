---
name: single-cell-integration-reproducibility-validation
description: Use when reproducing or auditing how STACAS weights integration anchors
  and derives the guide tree that orders dataset integration, or when validating an
  independent implementation against Run.STACAS defaults.
when_to_use_negative:
- Input data lacks prior cell type annotations and the goal is fully unsupervised
  integration — STACAS's semi-supervised label-confidence filtering of anchors is
  central to this skill
- The task is only evaluating integration quality (e.g., computing CiLISI/ASW with
  scIntegrationMetrics) rather than reconstructing the anchor weighting and tree logic
  itself
- Datasets are technical replicates where simple unsupervised methods (Harmony, FastMNN)
  suffice and anchor-based semi-supervised correction adds unnecessary label dependence
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: STACAS
  role: Source of FindAnchors.STACAS, FilterAnchors.STACAS, SampleTree.STACAS, and
    Run.STACAS; provides sampleObj example data and the reference guide tree/correction
    vectors to validate against
  repo: https://github.com/carmonalab/STACAS
- name: Seurat v4
  role: Underlying framework in which STACAS anchors are computed and the hierarchical
    integration tree is applied for Seurat integration
- name: scib-pipeline
  role: Benchmarking pipeline used to compare STACAS against other (semi-)supervised
    integration methods in the reproducibility assessment
  repo: https://github.com/carmonalab/scib-pipeline
- name: scIntegrationMetrics
  role: Provides CiLISI and ASW metrics used to judge whether the integration reproduced
    by the reconstructed tree preserves biological variability
  repo: https://github.com/carmonalab/scIntegrationMetrics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:a19b07e8345121f3d6687e8d344203994dc7d3a169c82b5d098a4a6477454c2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/STACAS
  tools:
  - STACAS
  - Seurat v4
  - scIntegrationMetrics
  - scib-pipeline
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-integration-reproducibility-validation

## Summary

Reconstructs and validates STACAS's anchor scoring and integration guide tree: anchors are weighted from pairwise rPCA distances via a logistic score, combined with SNN scores (alpha = 0.8), filtered by label confidence, and summed into dataset-pair similarities that yield a ward.D2 hierarchical tree determining integration order.

## When to use

Use when reproducing or auditing how STACAS weights integration anchors and derives the guide tree that orders dataset integration, or when validating an independent implementation against Run.STACAS defaults.

## When NOT to use

- Input data lacks prior cell type annotations and the goal is fully unsupervised integration — STACAS's semi-supervised label-confidence filtering of anchors is central to this skill
- The task is only evaluating integration quality (e.g., computing CiLISI/ASW with scIntegrationMetrics) rather than reconstructing the anchor weighting and tree logic itself
- Datasets are technical replicates where simple unsupervised methods (Harmony, FastMNN) suffice and anchor-based semi-supervised correction adds unnecessary label dependence

## Inputs

- sampleObj example dataset shipped with STACAS
- Pairwise integration anchors from FindAnchors.STACAS
- Pairwise rPCA distances computed on anchor sets
- Prior cell type labels with per-cell label probabilities
- Run.STACAS default output (guide tree and correction vectors) as reference

## Outputs

- Anchor score table with logistic-transformed rPCA scores, combined SNN/rPCA scores (alpha=0.8), and label-confidence rejections
- Dataset-pair global similarity matrix derived from summed anchor scores
- Guide tree (hclust ward.D2, rooted at the dataset with highest total anchor score) defining integration order
- stacas_anchor_weighting_validation.csv comparison table of independent vs Run.STACAS tree topology and correction vectors

## How to apply

Load the sampleObj example dataset shipped with STACAS and compute pairwise integration anchors with FindAnchors.STACAS, then filter them by pairwise rPCA distance using FilterAnchors.STACAS. Implement the anchor weighting independently: transform the rPCA distance to a score via a logistic function, combine rPCA and SNN scores with a geometric weighted log-sum using alpha = 0.8, and reject anchors whose cell type label probability falls below the label.confidence threshold (semi-supervised filtering). Build the guide tree by summing retained weighted anchor scores over dataset pairs to get global between-dataset similarities, cluster with hclust(method='ward.D2'), and root the tree at the dataset with the highest total anchor score, mirroring SampleTree.STACAS. Run Run.STACAS with default parameters on sampleObj to obtain the reference guide tree and integration correction vectors, then diff your independently computed tree topology and correction vectors against these defaults and write the comparison to stacas_anchor_weighting_validation.csv.

## Related tools

- **STACAS** (Source of FindAnchors.STACAS, FilterAnchors.STACAS, SampleTree.STACAS, and Run.STACAS; provides sampleObj example data and the reference guide tree/correction vectors to validate against) — https://github.com/carmonalab/STACAS
- **Seurat v4** (Underlying framework in which STACAS anchors are computed and the hierarchical integration tree is applied for Seurat integration)
- **scib-pipeline** (Benchmarking pipeline used to compare STACAS against other (semi-)supervised integration methods in the reproducibility assessment) — https://github.com/carmonalab/scib-pipeline
- **scIntegrationMetrics** (Provides CiLISI and ASW metrics used to judge whether the integration reproduced by the reconstructed tree preserves biological variability) — https://github.com/carmonalab/scIntegrationMetrics

## Examples

```
anchors <- FindAnchors.STACAS(sampleObj); anchors <- FilterAnchors.STACAS(anchors); tree <- SampleTree.STACAS(anchors); integrated <- Run.STACAS(sampleObj)
```

## Evaluation signals

- Independently derived guide tree topology (ward.D2, rooted at the dataset with highest summed anchor score) matches SampleTree.STACAS's tree from Run.STACAS defaults
- Logistic-transformed rPCA scores combined with SNN scores at alpha = 0.8 reproduce Run.STACAS's retained-anchor weighting and rejection pattern at label.confidence
- Correction vectors from the independent implementation match Run.STACAS's integration correction vectors within tolerance
- Comparison table stacas_anchor_weighting_validation.csv shows agreement of tree topology and vector values across implementations
- Integrated output preserves biological variance: semi-supervised STACAS should outperform unsupervised methods and scANVI/scGen per the published benchmark using CiLISI metrics

## Limitations

- STACAS relies on prior cell type labels; label-confidence filtering rejects anchors with inconsistent cell type labels, so anchors are unavailable for datasets lacking usable annotations
- STACAS is designed to be robust to incomplete and imprecise labels, but poorly annotated datasets may still reduce the number of retained anchors and distort the guide tree
- CiLISI-based evaluation requires cell type annotations, cannot be computed for cell types present in only one dataset, and is suboptimal for datasets with highly unequal cell numbers

## Evidence

- [methods] Anchor filtering and weighting by rPCA distance and labels: "`FilterAnchors.STACAS`  Filters integration anchors based on pairwise rPCA distance, calculated using `FindAnchors.STACAS`"
- [methods] Guide tree determination: "`SampleTree.STACAS`   Determine automatically a hierarchical tree for Seurat integration using anchors calculated in STACAS"
- [methods] Complete wrapper for the anchor-finding algorithm: "`Run.STACAS`   Wrapper function to run the complete anchor-finding algorithm in STACAS."
- [abstract] Rationale for semi-supervised anchor weighting: "STACAS, a batch correction method for scRNA-seq that leverages prior knowledge on cell types to preserve biological variability upon integration."
- [readme] Semi-supervised integration in downstream atlas construction confirms the workflow: [evidence span withheld — anchor status: not_found]
