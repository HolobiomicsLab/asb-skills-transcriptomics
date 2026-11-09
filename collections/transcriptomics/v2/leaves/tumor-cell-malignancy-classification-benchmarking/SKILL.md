---
name: tumor-cell-malignancy-classification-benchmarking
description: Use when scRNA-seq tumor samples with matched bulk WGS/WES copy-number
  ground truth are available and you need to quantify accuracy of CNA inference or
  malignant-cell classification against methods like SCEVAN, inferCNV, and CopyKAT.
when_to_use_negative:
- Tumors with minimal genomic alterations (leukemia, pediatric cancers, Ependymomas)
  where aneuploidy-based inference is unsuitable, so correlation vs bulk truth is
  not meaningful
- No matched bulk WGS/WES ground truth is available for the samples, making correlation-based
  benchmarking impossible
- Data are bulk-only or already segmented profiles with no single-cell expression
  matrix to run the inference tools on
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: SCEVAN
  role: Primary CNA inference and automatic malignant/non-malignant classification
    tool whose output (LogRatio segment means and clone labels) is benchmarked against
    bulk ground truth
  repo: github.com/AntonioDeFalco/SCEVAN
- name: CopyKAT
  role: Comparator method (v1.0.5) run with the SCEVAN workflow to obtain an alternative
    CNA profile for correlation benchmarking
- name: inferCNV
  role: Comparator method (v1.4.0) run with the SCEVAN workflow to obtain an alternative
    CNA profile for correlation benchmarking
- name: DNAcopy
  role: Segments the GBM bulk WGS ground truth (v1.62.0) used as reference for correlation
- name: CNVkit
  role: Segments the Multiple Myeloma bulk WES ground truth (v0.9.9) with integer
    cutoffs −1, −0.25, 0.2, 0.7
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:308cac8811f033bbdc86d767b8e97c47f2b5e0ab131372b4c46503d0701384c4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CNVkit
  - CopyKAT
  - DNAcopy
  - SCEVAN
  - inferCNV
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# tumor-cell-malignancy-classification-benchmarking

## Summary

Benchmarking protocol that quantifies how accurately scRNA-seq copy-number inference methods (SCEVAN, inferCNV, CopyKAT) reproduce bulk WGS/WES ground-truth CNA profiles by computing per-sample Pearson correlations on 1-Mb resampled segment means, and assesses malignant/non-malignant classification (F1 = 0.90 for SCEVAN vs 0.63 for state-of-the-art tools).

## When to use

Use when scRNA-seq tumor samples with matched bulk WGS/WES copy-number ground truth are available and you need to quantify accuracy of CNA inference or malignant-cell classification against methods like SCEVAN, inferCNV, and CopyKAT.

## When NOT to use

- Tumors with minimal genomic alterations (leukemia, pediatric cancers, Ependymomas) where aneuploidy-based inference is unsuitable, so correlation vs bulk truth is not meaningful
- No matched bulk WGS/WES ground truth is available for the samples, making correlation-based benchmarking impossible
- Data are bulk-only or already segmented profiles with no single-cell expression matrix to run the inference tools on

## Inputs

- Raw scRNA-seq count matrix (genes × cells) per tumor sample, e.g. GBM samples MGH102/MGH104/MGH105/MGH106 from GSE131928
- Matched bulk WGS/WES copy-number ground truth (Supplementary Data 4 for MM CNVkit segments; Supplementary Data 5 for GBM DNAcopy segments)
- Reference bulk segmentation files segmented with DNAcopy v1.62.0 or CNVkit v0.9.9

## Outputs

- Per-sample Pearson correlation values between each tool's inferred CNA profile and the 1-Mb resampled bulk truth
- Aggregated mean and maximum Pearson correlations per tool per cohort (GBM and MM)
- Summary table benchmark_correlations.csv
- Per-cell malignant/non-malignant classification and F1 scores (SCEVAN reports 0.90 vs 0.63 for state-of-the-art tools)

## How to apply

Run SCEVAN::pipelineCNA on each sample (e.g. the 26 multiregional GBM samples from GSE131928 and 7 Multiple Myeloma samples) with SUBCLONES = TRUE and par_cores = 20, and run CopyKAT v1.0.5 and inferCNV v1.4.0 on the same matrices following the SCEVAN workflow for comparability. Resample the bulk ground truth to 1-Mb bins to match single-cell resolution: GBM truth from low-depth WGS segmented with DNAcopy v1.62.0 (Supplementary Data 5) and MM truth from WES segmented with CNVkit v0.9.9 with integer cutoffs −1, −0.25, 0.2, and 0.7 (Supplementary Data 4). Compute Pearson correlations between each tool's inferred CNA profile (SCEVAN segment mean/LogRatio and binary classification, CopyKAT profile, inferCNV profile) and the resampled bulk truth per sample, then aggregate mean and max correlations across cohorts (SCEVAN LogRatio: mean 0.57, max 0.81 on GBM; mean 0.44, max 0.71 on MM). For classification accuracy, compare predicted malignant/non-malignant labels against available ground truth using F1 score. Rationale: bulk profiles have coarser resolution than single-cell data, so resampling to a common 1-Mb grid is required before correlation.

## Related tools

- **SCEVAN** (Primary CNA inference and automatic malignant/non-malignant classification tool whose output (LogRatio segment means and clone labels) is benchmarked against bulk ground truth) — github.com/AntonioDeFalco/SCEVAN
- **CopyKAT** (Comparator method (v1.0.5) run with the SCEVAN workflow to obtain an alternative CNA profile for correlation benchmarking)
- **inferCNV** (Comparator method (v1.4.0) run with the SCEVAN workflow to obtain an alternative CNA profile for correlation benchmarking)
- **DNAcopy** (Segments the GBM bulk WGS ground truth (v1.62.0) used as reference for correlation)
- **CNVkit** (Segments the Multiple Myeloma bulk WES ground truth (v0.9.9) with integer cutoffs −1, −0.25, 0.2, 0.7)

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- SCEVAN segment-mean (LogRatio) Pearson correlation vs bulk WGS should reach mean 0.57 (max 0.81) on the multiregional GBM cohort
- SCEVAN vs bulk WES ground truth on MM should reach mean 0.44 (max 0.71)
- Classification F1 across 106 samples / 93,332 cells should approximate 0.90 for SCEVAN vs 0.63 for state-of-the-art tools
- Bulk truth must be resampled to 1-Mb bins before correlation; correlations computed on unresampled profiles are invalid
- benchmark_correlations.csv must contain per-sample and aggregated per-tool mean/max correlations for all three tools on both cohorts

## Limitations

- Ground truth from WES/WGS has different resolution than single-cell data, requiring 1-Mb resampling before comparison, which limits benchmark fidelity
- Aneuploidy-based assumption is unsuitable for tumors with minimal genomic alterations (leukemia, pediatric cancers, Ependymomas)
- inferCNV-like methods require manual identification of reference cell clusters and perform sub-optimally on droplet-based data with very sparse coverage depth
- CopyKAT classification can be affected by wrong identification of normal cells and was not designed for complete automatic end-to-end clone identification

## Evidence

- [intro] Benchmark setup: "we benchmark the output of SCEVAN against state-of-the-art methods and show that SCEVAN exhibits faster and more accurate performance on synthetic and real data with reference copy number from bulk"
- [discussion] Comparator versions: "are analyzed following the steps of SCEVAN Workflow (Methods) and with CopyKAT v1.0.5 and inferCNV v1.4.0."
- [discussion] Resolution mismatch: "The ground truth extracted from WES and WGS have of course different resolutions with respect to the single-cell data."
- [supplementary] MM truth segmentation: "The integer Copy Number was assigned based on cutoffs specified in the CNVkit documentation (−1, −0.25, 0.2, and 0.7) (Supplementary Data 4)."
- [methods] Primary invocation: "results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)"
- [readme] F1 benchmark: [evidence span withheld — anchor status: external_doc]
