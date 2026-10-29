---
name: spatial-prediction-accuracy-evaluation-auc-rmse-spearman
description: Use when a method maps scRNA-seq cells to spatial positions and you must
  quantify how accurately held-out genes' spatial patterns are recovered against a
  spatial reference map.
when_to_use_negative:
- The mapping is evaluated on cell-type labels or spatial coordinates rather than
  held-out gene expression patterns — this skill only scores spatial gene expression
  reconstruction.
- No independent spatial reference measurements exist for the genes to be held out,
  so AUC/Spearman/RMSE against ground truth cannot be computed.
- The scRNA-seq data cannot be binarized or the spatial patterns are continuous with
  no meaningful binary state, making the AUC component inappropriate (Spearman RS
  and RMSE would still apply).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SpaOTsc
  role: Re-derives the cell-to-position optimal transport map for each leave-one-out
    fold and produces transport-plan-weighted spatial expression predictions to be
    scored
  repo: https://github.com/zcang/SpaOTsc
- name: scikit-learn
  role: Computes evaluation metrics (AUC, RMSE) on held-out gene predictions
- name: scipy
  role: Computes Spearman rank correlation (RS) between predicted and reference spatial
    patterns
- name: pandas
  role: Aggregates per-gene and per-dataset scores and writes the metrics CSV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:fa7f7d0180343a4e9450d76b948e3d7364c81926c3eb9fa96af12d8b60bb57f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zcang/SpaOTsc
  tools:
  - SpaOTsc
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Spatial prediction accuracy evaluation via AUC, Spearman correlation, and RMSE

## Summary

Evaluates spatial gene expression predictions (e.g., SpaOTsc mappings of zebrafish embryo scRNA-seq data onto a 64-position in situ hybridization reference map) by leave-one-out cross-validation over spatially measured genes, scoring each held-out gene with AUC (binarized patterns), Spearman rank correlation (RS), and RMSE. This reproduces the Table 1 validation of the Cang & Nie 2020 Nature Communications SpaOTsc method.

## When to use

Use when a method maps scRNA-seq cells to spatial positions and you must quantify how accurately held-out genes' spatial patterns are recovered against a spatial reference map.

## When NOT to use

- The mapping is evaluated on cell-type labels or spatial coordinates rather than held-out gene expression patterns — this skill only scores spatial gene expression reconstruction.
- No independent spatial reference measurements exist for the genes to be held out, so AUC/Spearman/RMSE against ground truth cannot be computed.
- The scRNA-seq data cannot be binarized or the spatial patterns are continuous with no meaningful binary state, making the AUC component inappropriate (Spearman RS and RMSE would still apply).

## Inputs

- Predicted spatial gene expression values at each of the 64 spatial positions for each held-out gene
- Spatial reference map (e.g., Spatial_ReferenceMap.xlsx, 64-position in situ hybridization map of 47 zebrafish genes)
- Binarized spatial gene expression patterns for AUC scoring
- Re-derived SpaOTsc transport plan per leave-one-out fold

## Outputs

- Per-gene AUC, Spearman rank correlation (RS), and RMSE scores
- Aggregated per-dataset metrics (e.g., zebrafish_loocv_metrics.csv with AUC, Spearman RS, RMSE columns)
- Comparison against reported reference values (Z. Em. 1 AUC 0.952, Z. Em. 2 AUC 0.936)

## How to apply

Hold out each spatially measured gene, re-derive the cell-to-position mapping without it (e.g., via spsc.transport_plan(cost_matrix) in SpaOTsc), then predict the held-out gene's expression at each spatial position as the transport-plan-weighted average of single-cell gene expression. Score each prediction with AUC after binarizing the spatial pattern, Spearman rank correlation (RS), and RMSE against the reference measurements. Repeat over all held-out genes and aggregate per-gene and per-dataset scores. For the zebrafish benchmark, expect AUC close to the reported Table 1 values of 0.952 (Z. Em. 1, GSE66688) and 0.936 (Z. Em. 2, GSE112294); write results with columns for AUC, Spearman RS, and RMSE to a metrics CSV for comparison.

## Related tools

- **SpaOTsc** (Re-derives the cell-to-position optimal transport map for each leave-one-out fold and produces transport-plan-weighted spatial expression predictions to be scored) — https://github.com/zcang/SpaOTsc
- **scikit-learn** (Computes evaluation metrics (AUC, RMSE) on held-out gene predictions)
- **scipy** (Computes Spearman rank correlation (RS) between predicted and reference spatial patterns)
- **pandas** (Aggregates per-gene and per-dataset scores and writes the metrics CSV)

## Examples

```
spsc.transport_plan(cost_matrix); pred = gamma @ df_sc[gene].values / gamma.sum(axis=1); from scipy.stats import spearmanr; rs, _ = spearmanr(pred, ref_spatial[gene])
```

## Evaluation signals

- Reproduced AUC for the two zebrafish datasets falls close to the reported Table 1 values of 0.952 (Z. Em. 1) and 0.936 (Z. Em. 2)
- Output metrics CSV exists with explicit columns for AUC, Spearman RS, and RMSE
- The larger second dataset (5693-cell, GSE112294) performs only slightly inferior to the first (851-cell, GSE66688), as reported
- Every spatially measured gene (47 genes on the 64-position map) is held out exactly once, and per-gene scores aggregate without missing folds
- Predictions match the scale of the reference map (RMSE near zero relative to binarized/normalized reference values)

## Limitations

- Evaluation is only as good as the spatial reference map; the zebrafish benchmark uses a small number of spatially measured genes (47) on 64 positions
- Binarization of scRNA-seq data (following the Seurat tutorial) and highly variable gene selection affect the AUC component of the score
- SpaOTsc computation can become intractable when the dataset is excessively large, limiting leave-one-out folds for large scRNA-seq datasets

## Evidence

- [abstract] Four datasets are employed for cross-validation of spatial gene expression prediction and comparison to known cell–cell communications.: "Four datasets are employed for cross-validation of spatial gene expression prediction and comparison to known cell–cell communications."
- [methods] We binarized the scRNA-seq data and selected a set of highly variable genes following the same tutorial.: "We binarized the scRNA-seq data and selected a set of highly variable genes following the same tutorial."
- [full_text] In leave-one-out cross-validation of spatial gene expression prediction, SpaOTsc achieved an AUC of 0.95 and 0.94 for the first (851-cell, GSE66688) and second (5693-cell, GSE112294) pairs of zebrafish embryo datasets, respectively.: [evidence span withheld — anchor status: not_found]
- [methods] SpaOTsc constructs a mapping between the n cells in scRNA-seq data and the m positions in spatial data by solving an optimal transport problem: "SpaOTsc constructs a mapping between the n cells in scRNA-seq data and the m positions in spatial data by solving an optimal transport problem"
- [discussion] the computation can become intractable when the dataset is excessively large: "the computation can become intractable when the dataset is excessively large"
