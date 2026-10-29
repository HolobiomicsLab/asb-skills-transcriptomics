---
name: tpr-fpr-benchmark-comparison
description: Use when comparing the sensitivity and specificity of two cell-classification
  stages on scRNA-seq data — e.g. testing whether scID's stage-3 weighted-score assignment
  raises the true-positive rate over stage-2 DPR training-cell selection without inflating
  false positives.
when_to_use_negative:
- Input expression matrices are log-transformed — scID requires library-depth (TPM/CPM)
  normalized, not log-transformed data
- Neither the reference nor target dataset is well clustered or contains good gene
  signatures — the article states scID is not a recommended approach in that case
- Reference and target are identical or near-duplicate datasets with no batch effect,
  where the DPR-vs-weighted-scoring comparison has no discriminative contrast to measure
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scID
  role: Provides find_markers (stage 1), choose_training_set and scID_weight (stage
    2), and final_populations (stage 3) whose TPR/FPR trade-off is benchmarked
  repo: https://github.com/BatadaLab/scID
- name: R
  role: Execution environment in which the scID library is loaded and the staged TPR/FPR
    comparison is computed and tabulated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:150319ca35fbcdf3c42517f969ad36e89e633004bf6866716345f314433b68fb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BatadaLab/scID
  repo_url: https://github.com/BatadaLab/scID
  tools:
  - R
  - scID
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# tpr-fpr-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that quantifies whether scID's stage-3 weight-based classification improves on stage-2 DPR-based cell selection by computing true-positive and false-positive rates separately for each stage on the same target dataset (e.g. mouse retinal bipolar Drop-seq data).

## When to use

Use when comparing the sensitivity and specificity of two cell-classification stages on scRNA-seq data — e.g. testing whether scID's stage-3 weighted-score assignment raises the true-positive rate over stage-2 DPR training-cell selection without inflating false positives.

## When NOT to use

- Input expression matrices are log-transformed — scID requires library-depth (TPM/CPM) normalized, not log-transformed data
- Neither the reference nor target dataset is well clustered or contains good gene signatures — the article states scID is not a recommended approach in that case
- Reference and target are identical or near-duplicate datasets with no batch effect, where the DPR-vs-weighted-scoring comparison has no discriminative contrast to measure

## Inputs

- Reference scRNA-seq gene expression matrix (genes x cells, library-depth normalized e.g. TPM/CPM), e.g. Shekhar et al. retinal bipolar Drop-seq reference
- Target scRNA-seq gene expression matrix (genes x cells, same normalization), e.g. mouse retinal bipolar Drop-seq target
- Named list of reference cluster labels for the reference cells

## Outputs

- Stage-2 selected positive/negative populations (DPR-based training cells)
- Stage-3 assigned target cell populations from final_populations(score)
- tpr_fpr_comparison.csv — side-by-side TPR and FPR for stage-2 vs stage-3 per cluster

## How to apply

Load the Shekhar et al. retinal bipolar Drop-seq reference and target gene expression matrices plus reference cluster labels in R, using library-depth normalized (TPM/CPM) values, not log-transformed data. Run scID::find_markers on the reference to extract cluster-specific gene signatures with logFC=0.6 as the minimum log-fold-change threshold. Execute scID stage 2 via scID::choose_training_set(gem, positive_markers, negative_markers) and scID::scID_weight(gem, true_cells, false_cells) to identify putative positive/negative training cells by differential precision-recall (DPR) and estimate discriminative gene weights, then execute stage 3 via scID::final_populations(score), which models target-cell scores with a finite mixture of Gaussians. Compute the true-positive rate (fraction of true cluster members retained) and false-positive rate (fraction of non-members included) separately for the stage-2 selected populations and the stage-3 assigned populations, tabulate them side by side, and save the result as tpr_fpr_comparison.csv. Accept the result if stage-3 shows a higher TPR than stage-2 while FPR is not increased, matching the Figure 1C pattern.

## Related tools

- **scID** (Provides find_markers (stage 1), choose_training_set and scID_weight (stage 2), and final_populations (stage 3) whose TPR/FPR trade-off is benchmarked) — https://github.com/BatadaLab/scID
- **R** (Execution environment in which the scID library is loaded and the staged TPR/FPR comparison is computed and tabulated)

## Examples

```
scID_output <- scID::scid_multiclass(target_gem = target_gem, reference_gem = reference_gem, reference_clusters = reference_clusters, estimate_weights_from_target = TRUE)
```

## Evaluation signals

- Stage-3 weight-based classification achieves a higher true-positive rate than stage-2 DPR selection
- Stage-3 false-positive rate is not increased relative to stage 2
- The tabulated pattern matches the Figure 1C comparison reported in Boufea et al. 2020
- tpr_fpr_comparison.csv contains both TPR and FPR columns for both stages and saves without error
- Marker extraction used the article's logFC=0.6 threshold and non-log-transformed (TPM/CPM) expression values

## Limitations

- Biomarker-based identification of equivalent cells across datasets does not work effectively owing to the dropout problem in scRNA-seq data
- If neither dataset is well clustered and contains good gene signatures, scID is not a recommended approach
- When the reference and target have strong batch effect, estimating gene signature weights from the target data appears more accurate than from the reference

## Evidence

- [readme] Negative markers improve classification: [evidence span withheld — anchor status: external_doc]
