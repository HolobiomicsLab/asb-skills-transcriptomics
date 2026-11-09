---
name: genomic-resolution-matching-resampling
description: Use when benchmarking scRNA-seq inferred copy number profiles against
  bulk WGS/WES ground truth whose native resolution differs from the single-cell profile
  bins, requiring harmonization before correlation.
when_to_use_negative:
- When the single-cell inferred profiles and the bulk ground truth are already on
  the same genomic bin grid — resampling is unnecessary
- When evaluating classification accuracy (e.g., malignant vs non-malignant F1 score)
  rather than continuous copy number profile concordance, since bin-matched profiles
  are not required
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: SCEVAN
  role: Produces the inferred single-cell CNA profiles (LogRatio, segment mean) that
    are correlated against the 1-Mb resampled bulk truth
  repo: https://github.com/AntonioDeFalco/SCEVAN
- name: CopyKAT v1.0.5
  role: Generates comparable copy number inference profiles for benchmarking against
    the resampled ground truth
- name: inferCNV v1.4.0
  role: Generates comparable copy number inference profiles for benchmarking against
    the resampled ground truth
- name: DNAcopy v1.62.0
  role: Segments the GBM low-depth WGS bulk ground truth prior to 1-Mb resampling
- name: CNVkit v0.9.9
  role: Segments the Multiple Myeloma WES bulk ground truth prior to 1-Mb resampling
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:6ed7520c00c89758e7b49f91390b57d316e6a72d3f64295b91751f989d79ceda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AntonioDeFalco/SCEVAN
  tools:
  - CNVkit v0.9.9
  - CopyKAT v1.0.5
  - DNAcopy v1.62.0
  - SCEVAN
  - inferCNV
  - inferCNV v1.4.0
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# genomic-resolution-matching-resampling

## Summary

Resamples segmented bulk DNA-sequencing copy number ground truth (WGS/WES) to 1-Mb bins so its resolution matches single-cell scRNA-seq inferred CNA profiles, enabling per-sample Pearson correlation benchmarks between tools like SCEVAN, CopyKAT, and inferCNV and the bulk truth.

## When to use

Use when benchmarking scRNA-seq inferred copy number profiles against bulk WGS/WES ground truth whose native resolution differs from the single-cell profile bins, requiring harmonization before correlation.

## When NOT to use

- When the single-cell inferred profiles and the bulk ground truth are already on the same genomic bin grid — resampling is unnecessary
- When evaluating classification accuracy (e.g., malignant vs non-malignant F1 score) rather than continuous copy number profile concordance, since bin-matched profiles are not required

## Inputs

- GBM bulk low-depth WGS segmented copy number data (DNAcopy v1.62.0, Supplementary Data 5)
- Multiple Myeloma bulk WES segmented copy number data (CNVkit v0.9.9, Supplementary Data 4)
- Per-cell inferred CNA profiles from SCEVAN pipelineCNA (LogRatio and binary classification)
- Per-cell inferred CNA profiles from CopyKAT v1.0.5 and inferCNV v1.4.0

## Outputs

- 1-Mb binned bulk ground-truth copy number tracks for each of the 26 GBM and 7 MM samples
- Per-sample Pearson correlation values between each tool's inferred profile and the resampled truth
- Summary table of mean and maximum Pearson correlations per tool per cohort (benchmark_correlations.csv)

## How to apply

Start from the segmented bulk ground truth: for the 26 GBM multiregional samples use low-depth WGS segmented with DNAcopy v1.62.0 (Supplementary Data 5), and for the 7 Multiple Myeloma samples use WES segmented with CNVkit v0.9.9 assigning integer copy number with cutoffs −1, −0.25, 0.2, and 0.7 (Supplementary Data 4). Resample these segments to 1-Mb bins so that every bulk truth track has the same genomic bin grid as the per-cell inferred CNA profiles from SCEVAN (LogRatio and binary classification), CopyKAT, and inferCNV. Then compute Pearson correlations per sample between each tool's inferred profile and the 1-Mb resampled truth, and aggregate mean and maximum correlations per cohort. The rationale is that WES/WGS ground truth has finer, gene-level resolution while single-cell expression-derived profiles are coarse; without matching bin grids the correlations would be ill-defined or biased.

## Related tools

- **SCEVAN** (Produces the inferred single-cell CNA profiles (LogRatio, segment mean) that are correlated against the 1-Mb resampled bulk truth) — https://github.com/AntonioDeFalco/SCEVAN
- **CopyKAT v1.0.5** (Generates comparable copy number inference profiles for benchmarking against the resampled ground truth)
- **inferCNV v1.4.0** (Generates comparable copy number inference profiles for benchmarking against the resampled ground truth)
- **DNAcopy v1.62.0** (Segments the GBM low-depth WGS bulk ground truth prior to 1-Mb resampling)
- **CNVkit v0.9.9** (Segments the Multiple Myeloma WES bulk ground truth prior to 1-Mb resampling)

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- Each resampled bulk truth track must contain exactly 1-Mb bins aligned to the genomic coordinate grid of the single-cell inferred profiles
- Per-sample Pearson correlation between SCEVAN LogRatio and bulk truth should fall near reported values: mean 0.57 (max 0.81) for the 26 GBM multiregional samples and mean 0.44 (max 0.71) for the 7 MM samples
- Integer copy number calls in the MM truth must reflect the CNVkit cutoffs −1, −0.25, 0.2, and 0.7 before resampling
- The summary table (benchmark_correlations.csv) must include mean and max Pearson correlations per tool for both the GBM and MM cohorts

## Limitations

- Ground truth extracted from WES and WGS has different resolution from single-cell data, so resampling introduces approximation and matching is imperfect by nature
- Aneuploidy-based inferred profiles are unsuitable for tumors with minimal genomic alterations (e.g., leukemia, pediatric cancers, Ependymomas), so correlations against bulk truth are not meaningful in those settings

## Evidence

- [discussion] The ground truth extracted from WES and WGS have of course different resolutions with respect to the single-cell data.: "The ground truth extracted from WES and WGS have of course different resolutions with respect to the single-cell data."
- [discussion] In the case of Multiple Myeloma17, CNVkit v0.9.9 was used for segmentation.: "In the case of Multiple Myeloma17, CNVkit v0.9.9 was used for segmentation."
- [discussion] was segmented using DNAcopy (v1.62.0)47 (Supplementary Data 5).: "was segmented using DNAcopy (v1.62.0)47 (Supplementary Data 5)."
- [discussion] The integer Copy Number was assigned based on cutoffs specified in the CNVkit documentation (−1, −0.25, 0.2, and 0.7) (Supplementary Data 4).: "The integer Copy Number was assigned based on cutoffs specified in the CNVkit documentation (−1, −0.25, 0.2, and 0.7) (Supplementary Data 4)."
- [intro] we benchmark the output of SCEVAN against state-of-the-art methods and show that SCEVAN exhibits faster and more accurate performance on synthetic and real data with reference copy number from bulk: "we benchmark the output of SCEVAN against state-of-the-art methods and show that SCEVAN exhibits faster and more accurate performance on synthetic and real data with reference copy number from bulk"
