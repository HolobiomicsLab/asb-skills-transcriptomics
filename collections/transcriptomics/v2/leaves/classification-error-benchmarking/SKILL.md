---
name: classification-error-benchmarking
description: Use when a selected or curated gene marker panel must be quantitatively
  compared against baselines by held-out cell label classification error in the marker-reduced
  feature space.
when_to_use_negative:
- The goal is marker selection itself rather than evaluating an already-selected panel
  — use scGeneFit's selection workflow instead.
- Cell type classes are suspected to be nonlinearly separable in the reduced marker
  space; the article notes linear separability is a key assumption reasonable only
  in high dimensions.
- The task involves continuous trajectories or spatial localization rather than categorical
  labels; the article notes scGeneFit supports only categorical labeling.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGeneFit
  role: Provides the jointly optimized marker panels whose discriminatory power is
    benchmarked by k-nearest-neighbor label classification error against curated and
    random panels.
  repo: github.com/solevillar/scGeneFit
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:caff9c46c5f667c1e13b154f3e6cdc62b7a5a312699c442425d467ce268024c3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGeneFit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# classification-error-benchmarking

## Summary

Benchmark a marker gene panel by the label classification error of a k-nearest-neighbor classifier trained in the reduced marker space, comparing the optimized panel (e.g., scGeneFit) against curated and random panels of matched size. This mirrors the evaluation used to validate scGeneFit on CBMC (8.05% vs 9.90% vs 30% error) and hierarchical Zeisel data.

## When to use

Use when a selected or curated gene marker panel must be quantitatively compared against baselines by held-out cell label classification error in the marker-reduced feature space.

## When NOT to use

- The goal is marker selection itself rather than evaluating an already-selected panel — use scGeneFit's selection workflow instead.
- Cell type classes are suspected to be nonlinearly separable in the reduced marker space; the article notes linear separability is a key assumption reasonable only in high dimensions.
- The task involves continuous trajectories or spatial localization rather than categorical labels; the article notes scGeneFit supports only categorical labeling.

## Inputs

- Normalized scRNA-seq expression matrix (log2(1+X) transform), e.g., CBMC CITE-seq from GEO GSE100866 or Zeisel GSE60361
- Cell type label vector, optionally with a two-layer label hierarchy annotation
- Selected marker sets: scGeneFit panel, curated marker panel, and a size-matched random marker panel

## Outputs

- CSV table with one label classification error entry per marker set (e.g., 8.05% scGeneFit, 9.90% curated, 30% random on CBMC)
- First-layer hierarchy classification error table for hierarchical benchmarks (e.g., 8.75% scGeneFit-30, 6.44% scGeneFit-40, 9.74% one-vs-all-48)
- Selected marker gene lists per panel written to CSV

## How to apply

Normalize the expression matrix with a log2(1+X) transform (as done for the CBMC CITE-seq data from GEO accession GSE100866), then assemble the marker panels to compare: the jointly optimized scGeneFit panel for the target panel size, a curated panel (e.g., the curated CBMC marker set), and a random panel matched in size. Split cells into a 70%/30% train/test partition and, separately for each panel, train a k-nearest-neighbor classifier on the 70% split using only the marker genes as features. Evaluate label classification error on the held-out 30%. For hierarchical label structures (e.g., the two-layer Zeisel GSE60361 hierarchy), classify at the first layer and compare against a one-vs-all baseline of equal or larger size (e.g., 30/40 scGeneFit markers vs 48 one-vs-all markers). Validation requires one error entry per panel and the scGeneFit error to be lowest, matching the reported ordering.

## Related tools

- **scGeneFit** (Provides the jointly optimized marker panels whose discriminatory power is benchmarked by k-nearest-neighbor label classification error against curated and random panels.) — github.com/solevillar/scGeneFit

## Evaluation signals

- The emitted CSV contains one error entry per marker set (scGeneFit, curated, random) with no missing rows.
- On CBMC held-out data, scGeneFit error is lowest, reproducing the ordering and values 8.05% (scGeneFit) < 9.90% (curated) < 30% (random).
- For the Zeisel hierarchy benchmark, first-layer errors land near 8.75% (30 markers), 6.44% (40 markers), and 9.74% (one-vs-all 48 markers), showing scGeneFit with fewer markers on par with or better than one-vs-all.
- Random and curated panels are matched in size to the scGeneFit panel so error comparisons are fair.
- Train/test split is fixed at 70%/30% and the classifier is trained only on marker-space features.

## Limitations

- A key underlying assumption is that cell type classes are linearly separable, which is reasonable in high dimensions but may not hold in the reduced marker space.
- No nonlinear dimension reduction methods were implemented; relaxing the linear separability assumption was planned future work.
- Marker selection and evaluation are limited to categorical labels; placing unlabeled cells on trajectories or in spatial assays is not supported.

## Evidence

- [abstract] scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods.: "scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods."
- [results] kNN evaluation on CBMC: "we applied scGeneFit to a cord blood mononuclear cell (CBMC) study containing isolated cells from cord blood"
- [discussion] Linear separability assumption: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not"
- [intro] Marker selection scope: "gene panels must be designed that allow us to distinguish between cell labels both efficiently (with as few markers as possible) and at high precision"
