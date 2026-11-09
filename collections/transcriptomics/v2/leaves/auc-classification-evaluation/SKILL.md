---
name: auc-classification-evaluation
description: Use when you have per-cell gene set scores and a binary cell-level label
  (treated/control, injured/control, cell-type vs rest) and need a single threshold-free
  metric of class separation, typically to test whether nearest-neighbor smoothing
  improved discrimination.
when_to_use_negative:
- The task is gene set scoring or smoothing itself rather than quantifying class separation
  — use gssnng scoring/smoothing workflows without the AUC evaluation step
- There are more than two classes and no natural binary grouping; ROC AUC as applied
  here is defined for two-class discrimination (e.g., B cells vs non-B cells, injured
  vs control)
- Scores are not comparable across cells because they were computed with different
  gene sets, score methods, or groupby graphs without a controlled comparison
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: gssnng
  role: Produces the per-cell gene set score vectors (summed_up, singscore, ssGSEA,
    rank_biased_overlap) under smoothed and unsmoothed conditions that are then evaluated
    by AUC
  repo: https://github.com/IlyaLab/gssnng
- name: Scanpy
  role: Loads the h5ad AnnData objects (e.g., sc.datasets.pbmc3k_processed()) and
    supplies adata.obs class labels used for AUC computation
- name: AnnData
  role: Container format (h5ad) holding the cell-by-gene counts, neighbor graph, and
    per-cell labels and scores used in the AUC analysis
- name: Python
  role: Implementation language for computing ROC AUCs and writing the results CSV
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:35f5a0010f4f0c9b3246ff02181c30ff42a5a96497f5e85e8e5e55a9d5f9ed13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IlyaLab/gssnng
  tools:
  - AnnData
  - Python
  - Scanpy
  - gssnng
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# auc-classification-evaluation

## Summary

Compute ROC AUC of per-cell gene set scores (produced with gssnng scoring methods such as summed_up, singscore, ssGSEA, or rank_biased_overlap) to quantify how well a signature separates two cell classes (e.g., LPS-treated vs untreated phagocytes, injured vs control endothelial cells), and compare AUCs across smoothing conditions.

## When to use

Use when you have per-cell gene set scores and a binary cell-level label (treated/control, injured/control, cell-type vs rest) and need a single threshold-free metric of class separation, typically to test whether nearest-neighbor smoothing improved discrimination.

## When NOT to use

- The task is gene set scoring or smoothing itself rather than quantifying class separation — use gssnng scoring/smoothing workflows without the AUC evaluation step
- There are more than two classes and no natural binary grouping; ROC AUC as applied here is defined for two-class discrimination (e.g., B cells vs non-B cells, injured vs control)
- Scores are not comparable across cells because they were computed with different gene sets, score methods, or groupby graphs without a controlled comparison

## Inputs

- Per-cell gene set score vector from gssnng scoring (e.g., adata column or obsm output from score_cells.with_gene_sets)
- Binary per-cell class labels in adata.obs (e.g., LPS-treated vs untreated, injured vs control endothelial cells, naive B cells vs other PBMC types)
- Score vectors from multiple smoothing conditions (smooth_mode='off' vs smooth_mode='connectivity' with recompute_neighbors of 4, 8, or 32)
- Downsampling-fraction series (95% to 5% of total counts) when evaluating robustness to sequencing-depth reduction

## Outputs

- ROC AUC value per scoring method and smoothing condition
- CSV table with one labeled row per condition, e.g., lps_smoothing_auc_results.csv or auc_vs_downsampling.csv
- AUC-versus-downsampling-fraction comparison of smoothed vs non-smoothed conditions

## How to apply

After scoring cells with gssnng (e.g., score_method='summed_up' with ranked=True on the Okumura Inflammatory Response LPS gene set, or 'rank_biased_overlap' and 'ssGSEA' with method_params={'omega': 0.75} on the 12-gene _UP/_DN injury signature), collect the score vector per cell and the binary label from adata.obs. For each condition (smooth_mode='off' vs smooth_mode='connectivity' at recompute_neighbors=4, 8, or 32), compute the ROC AUC of the score classifying the two classes. Assemble one labeled row per condition into a CSV such as lps_smoothing_auc_results.csv or auc_vs_downsampling.csv. Judge success by comparing smoothed vs unsmoothed AUCs: the paper reports LPS summed_up AUC rising from 0.95 (unsmoothed) to 1.0 (smoothed), injury-signature AUCs rising from 0.62 to 0.71 (RBO) and 0.69 to 0.8 (ssGSEA), and K=32-smoothed B-cell scores holding AUC above 0.95 until fewer than 10% of counts remain under downsampling.

## Related tools

- **gssnng** (Produces the per-cell gene set score vectors (summed_up, singscore, ssGSEA, rank_biased_overlap) under smoothed and unsmoothed conditions that are then evaluated by AUC) — https://github.com/IlyaLab/gssnng
- **Scanpy** (Loads the h5ad AnnData objects (e.g., sc.datasets.pbmc3k_processed()) and supplies adata.obs class labels used for AUC computation)
- **AnnData** (Container format (h5ad) holding the cell-by-gene counts, neighbor graph, and per-cell labels and scores used in the AUC analysis)
- **Python** (Implementation language for computing ROC AUCs and writing the results CSV)

## Examples

```
from sklearn.metrics import roc_auc_score; auc = roc_auc_score(labels, scores)  # labels: 1=LPS-treated/injured/B-cell, 0=control; scores from gssnng score_cells.with_gene_sets
```

## Evaluation signals

- Unsmoothed AUC approximates the reported baseline (e.g., 0.95 for LPS summed_up scores, 0.62 for RBO and 0.69 for ssGSEA on endothelial injury scores)
- Smoothed AUC >= unsmoothed AUC for each scorer, with reported targets of approximately 1.0 (LPS), 0.71 (RBO), and 0.8 (ssGSEA)
- Under downsampling, the K=32 smoothed curve maintains AUC above 0.95 until the retained count fraction drops below 10%, while the non-smoothed curve degrades at higher fractions
- Output CSV contains one labeled row per condition (e.g., off, K=4, K=8, K=32; or unsmoothed/smoothed x RBO/ssGSEA) with all expected combinations present
- AUC values lie in [0, 1] and labels in the CSV unambiguously map each value to its score method and smoothing condition

## Limitations

- AUC measures two-class separation only; gssnng scores also support per-group visualization and statistical analysis beyond this binary evaluation
- Smoothing gains depend on building the nearest neighbor graph within meaningful groups (groupby), and on recompute_neighbors being rebuilt within each group; results may vary with the K chosen (4, 8, 32)
- Smoothed scores are mini-pseudobulk profiles shared across neighbors, so single-cell resolution of the score reflects neighborhood composition rather than purely individual-cell expression

## Evidence

- [other] Smoothing improved the AUC for distinguishing LPS-treated from untreated phagocytes using the summed_up ranked score: [evidence span withheld — anchor status: not_found]
- [readme] The GSSNNG method is based on using the nearest neighbor graph of cells for data smoothing. This essentially creates mini-pseudobulk expression profiles for each cell: "The GSSNNG method is based on using the nearest neighbor graph of cells for data smoothing. This essentially creates "
- [readme] smooth_mode: "adjacency", "connectivity", or "off" — Dictates how to use the neighborhood graph: "smooth_mode:** "adjacency", "connectivity", or "off"<br/>"
- [other] with 32-neighbor smoothing the AUC for distinguishing B cells from non-B cells stays above 0.95 until fewer than 10% of total counts remain: [evidence span withheld — anchor status: not_found]
