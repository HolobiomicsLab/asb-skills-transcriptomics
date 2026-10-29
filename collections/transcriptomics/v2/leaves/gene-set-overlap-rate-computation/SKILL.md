---
name: gene-set-overlap-rate-computation
description: Use when evaluating whether a feature-selection strategy (Seurat-style
  HVG, random-forest HIG, or scCAD ensemble) preserves a reference rare-cell-type
  DE gene set, using OR1/OR2/OR3 overlap rates.
when_to_use_negative:
- The goal is rare cell type detection itself rather than evaluating feature selection
  — use scCAD's full cluster-decomposition pipeline instead
- No annotated rare cell types exist to define a reference DE gene set S_ref via Seurat
  FindMarkers
- The candidate gene sets come from different gene identifiers/annotations that have
  not been harmonized before intersecting
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scCAD
  role: Provides the ensemble feature-selection strategy (merge and de-duplicate top
    2000 HVG with top 2000 random-forest-importance HIG) whose preserved genes are
    scored by the overlap rates
  repo: https://github.com/xuyp-csu/scCAD
- name: Seurat
  role: FindMarkers (Wilcoxon rank sum, FDR 0.05, |FC| cutoff 1.5) builds the reference
    DE gene set S_ref; Seurat-style mean-vs-dispersion HVG selection provides one
    comparison strategy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:1c7efac7664064d160dedddc47683d341c06ddf8ead90776a20ae64106c4dc83
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xuyp-csu/scCAD
  repo_url: https://github.com/xuyp-csu/scCAD
  tools:
  - Seurat
  - scCAD
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gene-set-overlap-rate-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes three overlap rates (OR1 = |S_ref ∩ S_select|/|S_ref|, OR2 = |S_ref ∩ S_select|/|S_ref ∪ S_select|, OR3 = |S_ref ∩ S_select|/|S_select|) between a reference rare-cell-type DE gene set and feature-selected gene sets (top 2000 HVG, top 2000 HIG, and the scCAD ensemble of both), to quantify how well each selection strategy preserves rare cell type-specific genes.

## When to use

Use when evaluating whether a feature-selection strategy (Seurat-style HVG, random-forest HIG, or scCAD ensemble) preserves a reference rare-cell-type DE gene set, using OR1/OR2/OR3 overlap rates.

## When NOT to use

- The goal is rare cell type detection itself rather than evaluating feature selection — use scCAD's full cluster-decomposition pipeline instead
- No annotated rare cell types exist to define a reference DE gene set S_ref via Seurat FindMarkers
- The candidate gene sets come from different gene identifiers/annotations that have not been harmonized before intersecting

## Inputs

- Preprocessed scRNA-seq expression matrices (cells × genes) with rare cell-type annotations per benchmark dataset
- Rare cell-type cluster/label assignments used to define reference DE genes
- Feature-selected gene sets: top 2000 HVG, top 2000 HIG (random forest Gini importance), and the de-duplicated scCAD ensemble of both

## Outputs

- Per-dataset OR1, OR2, OR3 values for each selection strategy (HVG, HIG, scCAD ensemble)
- feature_selection_overlap_rates.csv with per-strategy averages (expected: scCAD ≈ 86.75% OR1, HIG ≈ 80.20%, HVG ≈ 67.80%)

## How to apply

Preprocess each benchmark scRNA-seq dataset by retaining genes expressed in at least three cells and applying log-normalization with cell-specific size factors followed by log2(x+1) transformation. Build the reference set S_ref by finding the top 50 differentially expressed genes per rare cell type with Seurat FindMarkers (Wilcoxon rank sum test, FDR cutoff 0.05, absolute fold-change cutoff 1.5) and pooling them. Produce three selection sets: Seurat-style top 2000 highly variable genes (mean vs dispersion), top 2000 highly important genes from a scikit-learn random forest with Gini impurity importance trained on cluster labels, and the scCAD ensemble formed by merging and de-duplicating both top-2000 sets. For each strategy compute OR1 = |S_ref ∩ S_select|/|S_ref|, OR2 = |S_ref ∩ S_select|/|S_ref ∪ S_select|, and OR3 = |S_ref ∩ S_select|/|S_select|, then aggregate per-dataset values into per-strategy averages and write feature_selection_overlap_rates.csv. OR1 measures recovery of known rare-type DE genes, OR2 balances over-selection (Jaccard-style), and OR3 measures precision of the selected set.

## Related tools

- **scCAD** (Provides the ensemble feature-selection strategy (merge and de-duplicate top 2000 HVG with top 2000 random-forest-importance HIG) whose preserved genes are scored by the overlap rates) — https://github.com/xuyp-csu/scCAD
- **Seurat** (FindMarkers (Wilcoxon rank sum, FDR 0.05, |FC| cutoff 1.5) builds the reference DE gene set S_ref; Seurat-style mean-vs-dispersion HVG selection provides one comparison strategy)

## Examples

```
OR1 = len(S_ref & S_hvg)/len(S_ref); OR2 = len(S_ref & S_hvg)/len(S_ref | S_hvg); OR3 = len(S_ref & S_hvg)/len(S_hvg)
```

## Evaluation signals

- Reproduced average OR1 values match Supplementary Table 8: scCAD ≈ 86.75%, HVG ≈ 67.80%, HIG ≈ 80.20%
- All overlap rates fall in (0, 1]; OR2 must be ≤ min(OR1, OR3) since the union denominator is ≥ each individual set size
- The scCAD ensemble set is exactly the de-duplicated union of the top 2000 HVG and top 2000 HIG sets (size between 2000 and 4000)
- feature_selection_overlap_rates.csv contains one OR1/OR2/OR3 row per dataset per strategy plus per-strategy averages
- S_ref size equals the number of unique genes after pooling top 50 DE genes per rare cell type across all rare types in a dataset

## Limitations

- The reference DE set depends on Seurat FindMarkers parameters (top 50 per rare type, FDR cutoff 0.05, absolute fold-change cutoff 1.5); changing these changes the denominators of OR1/OR2/OR3
- Preprocessing thresholds (genes expressed in at least three cells, log2(x+1) after size-factor normalization) directly affect both S_ref and the HVG/HIG rankings
- Supplementary Table 8 with the reference figures is not included in the main article text, so validation relies on the values cited in the task card (≈86.75%/67.80%/80.20%)
- Results require reliable rare-type annotations; mislabeled or absent rare populations make S_ref uninformative

## Evidence

- [discussion] Ensemble feature selection rationale: [evidence span withheld — anchor status: not_found]
