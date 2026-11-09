---
name: gene-signature-based-training-label-calibration
description: Use when building or retraining a malignant-vs-normal scRNA-seq classifier
  and the initial malignancy labels come from heterogeneous original annotations that
  may contain mislabeled cells.
when_to_use_negative:
- When the input training labels are already expert-curated and signature-consistent
  — the calibration step removes ~2.5% of cells and would discard valid data
- When the task is unsupervised CNV-based malignancy inference (e.g., CopyKAT/inferCNVpy)
  rather than supervised classifier training on labeled data
- When annotating cell types broadly rather than distinguishing malignant from originating
  normal cells within a cancerous lineage — the method requires prior basic cell-type
  identification
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMalignantFinder
  role: Provides the signature-calibration machinery (aucell_cal on curated cancer
    gene signatures) used to refine training labels
  repo: github.com/Jonyyqn/scMalignantFinder
- name: Cancer-Finder
  role: Downstream supervised classifier retrained on the signature-calibrated training
    set to benchmark label-calibration benefit
  repo: github.com/Patchouli-M/SequencingCancerFinder
- name: ikarus
  role: Downstream logistic classifier whose gene signature and training labels are
    replaced by the calibrated set and DEG feature list
  repo: github.com/BIMSBbioinfo/ikarus
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:ead64723cb0c3aa1a9040c65e76e288677f26e42473424d81e3a559656ad109f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Jonyyqn/scMalignantFinder
  tools:
  - Cancer-Finder
  - ikarus
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# gene-signature-based-training-label-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Refine noisy training labels for malignant-cell classifiers by scoring cells against nine curated pan-cancer cancer gene signatures (e.g., via AUCell on sc_malignant_deg.gmt) and excluding cells that lose their expected signature profile, yielding a signature-calibrated training set that measurably improves downstream supervised classifiers such as Cancer-Finder and ikarus.

## When to use

Use when building or retraining a malignant-vs-normal scRNA-seq classifier and the initial malignancy labels come from heterogeneous original annotations that may contain mislabeled cells.

## When NOT to use

- When the input training labels are already expert-curated and signature-consistent — the calibration step removes ~2.5% of cells and would discard valid data
- When the task is unsupervised CNV-based malignancy inference (e.g., CopyKAT/inferCNVpy) rather than supervised classifier training on labeled data
- When annotating cell types broadly rather than distinguishing malignant from originating normal cells within a cancerous lineage — the method requires prior basic cell-type identification

## Inputs

- Annotated scRNA-seq training matrix in .h5ad format (combine_training.h5ad) with labels in adata.obs['Raw_annotation'] ('Normal'/'Malignant'/'Tumor')
- Curated cancer gene signatures in .gmt format (sc_malignant_deg.gmt, nine pan-cancer signature sets)
- DEG feature list (combined_tumor_up_down_degs.txt) used for downstream model retraining

## Outputs

- Signature-calibrated training set with refined malignancy labels and ~2.5% of mislabeled cells excluded
- Per-cell malignancy/signature scores used for calibration
- Retrained classifier (e.g., Cancer-Finder or ikarus) and benchmark comparison table (e.g., retrained_classifier_benchmark.csv) with accuracy and balanced accuracy per dataset

## How to apply

Start from an annotated scRNA-seq training set (e.g., combine_training.h5ad with labels in adata.obs['Raw_annotation'] carrying values 'Normal', 'Malignant', 'Tumor') whose malignant calls derive from varied methodologies in the original studies. Score every cell against the nine carefully curated pan-cancer gene signatures (sc_malignant_deg.gmt from Zenodo record 17888140) using the signature-scoring step provided by the method (adata = utils.aucell_cal(adata, sc_gmt)), then exclude the ~2.5% of initially annotated malignant cells that lost their upregulated/downregulated cancer signatures during calibration — these are treated as mislabeled. Keep the remaining signature-consistent cells as the calibrated training labels and normalize counts with sc.pp.normalize_total(adata, target_sum=1e4) (no internal log-transformation for Cancer-Finder-style models). Retrain the target classifier on this calibrated set using the union-of-DEGs feature list (combined_tumor_up_down_degs.txt); the rationale is that signature consistency filters label noise, which otherwise compromises supervised methods. Validate by re-evaluating on the same 13 benchmark datasets: retrained Cancer-Finder should improve from accuracy 0.654 to 0.828 and balanced accuracy 0.531 to 0.669, and ikarus from 0.446/0.533 to 0.488/0.612.

## Related tools

- **scMalignantFinder** (Provides the signature-calibration machinery (aucell_cal on curated cancer gene signatures) used to refine training labels) — github.com/Jonyyqn/scMalignantFinder
- **Cancer-Finder** (Downstream supervised classifier retrained on the signature-calibrated training set to benchmark label-calibration benefit) — github.com/Patchouli-M/SequencingCancerFinder
- **ikarus** (Downstream logistic classifier whose gene signature and training labels are replaced by the calibrated set and DEG feature list) — github.com/BIMSBbioinfo/ikarus

## Examples

```
adata = sc.pp.normalize_total(adata, target_sum=1e4); adata = utils.aucell_cal(adata, sc_gmt)
```

## Evaluation signals

- Label-calibration invariant: roughly 2.5% of initially annotated malignant cells are excluded for losing upregulated/downregulated cancer signatures (Supplementary Fig. 2a); a far larger or smaller exclusion fraction indicates misapplied scoring
- Calibrated malignant cells should carry higher malignancy probability than the excluded 2.5% of cells
- Retrained Cancer-Finder on the calibrated set must reach average accuracy 0.828 and balanced accuracy 0.669 across the 13 benchmark datasets
- Retrained ikarus must reach accuracy 0.488 and balanced accuracy 0.612; label space must remain binary 'Normal'/'Malignant'
- Schema check: training labels present in adata.obs['Raw_annotation'] with exactly the values 'Normal', 'Malignant', and 'Tumor'; counts normalized with target_sum=1e4 and no log-transformation before Cancer-Finder

## Limitations

- Excluding the 2.5% signature-inconsistent cells assumes label noise manifests as signature loss; cancers with unknown or controversial origins (e.g., synovial sarcoma, certain brain tumors) may not be captured by the nine curated signatures
- The calibrated training set is pan-cancer-derived, so calibration may weaken classifier performance on cancer types with atypical signature patterns or substantial inter-tumor heterogeneity
- The tool presupposes prior identification of basic cell types; calibration and downstream malignant identification are restricted to cancerous lineages

## Evidence
