---
name: copy-number-profile-correlation-with-bulk-truth
description: Use when scRNA-seq copy number inferences (SCEVAN, CopyKAT, inferCNV)
  need quantitative validation against matched bulk WGS or WES segmented profiles,
  e.g. for benchmarking tools on GBM or Multiple Myeloma cohorts.
when_to_use_negative:
- Tumors with minimal genomic alterations (e.g. leukemia, pediatric cancers, Ependymomas)
  where aneuploidy-based CNA inference is unsuited and correlation with bulk truth
  is not meaningful
- No matched bulk WGS/WES ground truth exists for the samples, so the correlation
  benchmark cannot be computed
- The goal is cell-level malignant/non-malignant classification accuracy (F1) rather
  than profile-level agreement with bulk segmentation
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCEVAN
  role: Infers per-cell copy number profiles and malignant/non-malignant classification
    via pipelineCNA; its LogRatio (segment mean) profile is the primary output benchmarked
    against bulk truth
  repo: https://github.com/AntonioDeFalco/SCEVAN
- name: CopyKAT v1.0.5
  role: Comparison method producing copy number profiles correlated against the same
    bulk ground truth under the SCEVAN Workflow
- name: inferCNV v1.4.0
  role: Comparison method producing copy number profiles correlated against the same
    bulk ground truth under the SCEVAN Workflow
- name: DNAcopy v1.62.0
  role: Segments the low-depth WGS GBM bulk ground truth (Supplementary Data 5) used
    for 1-Mb resampling
- name: CNVkit v0.9.9
  role: Segments the WES Multiple Myeloma bulk ground truth (Supplementary Data 4)
    with integer copy number cutoffs −1, −0.25, 0.2, 0.7
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:da052add1d39b0ed1f8802454e0fa4a1a2402dda5b99af2216253b54dc1a5ca9
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# copy-number-profile-correlation-with-bulk-truth

## Summary

Benchmarks copy number profiles inferred from scRNA-seq by SCEVAN, CopyKAT v1.0.5, and inferCNV v1.4.0 against bulk DNA sequencing ground truth by computing Pearson correlations on 1-Mb resampled profiles. It quantifies how faithfully single-cell CNA inference recovers bulk WGS/WES segmentations.

## When to use

Use when scRNA-seq copy number inferences (SCEVAN, CopyKAT, inferCNV) need quantitative validation against matched bulk WGS or WES segmented profiles, e.g. for benchmarking tools on GBM or Multiple Myeloma cohorts.

## When NOT to use

- Tumors with minimal genomic alterations (e.g. leukemia, pediatric cancers, Ependymomas) where aneuploidy-based CNA inference is unsuited and correlation with bulk truth is not meaningful
- No matched bulk WGS/WES ground truth exists for the samples, so the correlation benchmark cannot be computed
- The goal is cell-level malignant/non-malignant classification accuracy (F1) rather than profile-level agreement with bulk segmentation

## Inputs

- Raw scRNA-seq count matrices (genes × cells) for each tumor sample
- Bulk WGS ground-truth segmentation for 26 multiregional GBM samples (Supplementary Data 5, DNAcopy v1.62.0)
- Bulk WES ground-truth segmentation for 7 Multiple Myeloma samples (Supplementary Data 4, CNVkit v0.9.9, integer cutoffs −1, −0.25, 0.2, 0.7)

## Outputs

- Per-sample Pearson correlations between each tool's inferred CNA profile (SCEVAN LogRatio, CopyKAT, inferCNV) and the 1-Mb resampled bulk truth
- Aggregated per-tool mean and maximum Pearson correlations across the GBM and MM cohorts
- benchmark_correlations.csv summary table

## How to apply

Run SCEVAN::pipelineCNA on each sample (SUBCLONES = TRUE, par_cores = 20) alongside CopyKAT v1.0.5 and inferCNV v1.4.0, following the SCEVAN Workflow (Methods) so all tools are applied comparably. Resample the bulk ground truth to 1-Mb bins to match single-cell resolution, since WES/WGS profiles have different resolution than single-cell data: for the 26 multiregional GBM samples use low-depth WGS segmented with DNAcopy v1.62.0 (Supplementary Data 5), and for the 7 Multiple Myeloma samples use WES segmented with CNVkit v0.9.9 with integer copy number cutoffs −1, −0.25, 0.2, and 0.7 (Supplementary Data 4). For SCEVAN, use the segment mean (LogRatio) profile for correlation and optionally the binary malignant/non-malignant classification. Compute Pearson correlations between each tool's profile and the 1-Mb resampled truth per sample, then aggregate mean and maximum correlations across the cohort (reported reference values: SCEVAN mean 0.57, max 0.81 on GBM vs bulk WGS; mean 0.44, max 0.71 on MM vs WES) and export as benchmark_correlations.csv.

## Related tools

- **SCEVAN** (Infers per-cell copy number profiles and malignant/non-malignant classification via pipelineCNA; its LogRatio (segment mean) profile is the primary output benchmarked against bulk truth) — https://github.com/AntonioDeFalco/SCEVAN
- **CopyKAT v1.0.5** (Comparison method producing copy number profiles correlated against the same bulk ground truth under the SCEVAN Workflow)
- **inferCNV v1.4.0** (Comparison method producing copy number profiles correlated against the same bulk ground truth under the SCEVAN Workflow)
- **DNAcopy v1.62.0** (Segments the low-depth WGS GBM bulk ground truth (Supplementary Data 5) used for 1-Mb resampling)
- **CNVkit v0.9.9** (Segments the WES Multiple Myeloma bulk ground truth (Supplementary Data 4) with integer copy number cutoffs −1, −0.25, 0.2, 0.7)

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- Ground truth and inferred profiles must be resampled to the same 1-Mb bins before correlation is computed
- SCEVAN GBM cohort result should reproduce mean Pearson r ≈ 0.57 (max ≈ 0.81) vs low-depth WGS truth
- SCEVAN MM cohort result should reproduce mean Pearson r ≈ 0.44 (max ≈ 0.71) vs WES truth
- Per-sample correlation matrix must be complete for all 26 GBM and 7 MM samples across SCEVAN, CopyKAT, and inferCNV
- benchmark_correlations.csv schema: tool, sample, cohort, pearson_r, plus cohort-level mean/max aggregates

## Limitations

- Ground truth from WES/WGS has different resolution than single-cell data, requiring 1-Mb resampling before comparison; residual resolution mismatch can bias correlations
- Aneuploidy-based inference (and hence this benchmark) may not be suited to tumors with minimal genomic alterations such as leukemia, pediatric cancers, and Ependymomas
- The β constant (the only segmentation parameter) and initial mixture model parameters (Supplementary Table 1) are not fully specified in the main text, complicating exact reproduction

## Evidence

- [abstract] SCEVAN as segment mean (LogRatio) benchmarked: "SCEVAN can automatically and accurately discriminate between malignant and non-malignant cells, resulting in a practical framework to analyze tumors"
- [discussion] Comparison tool versions under SCEVAN workflow: "are analyzed following the steps of SCEVAN Workflow (Methods) and with CopyKAT v1.0.5 and inferCNV v1.4.0."
- [discussion] Resolution mismatch rationale for resampling: "The ground truth extracted from WES and WGS have of course different resolutions with respect to the single-cell data."
- [discussion] MM bulk ground truth segmentation tool: "In the case of Multiple Myeloma17, CNVkit v0.9.9 was used for segmentation."
- [discussion] GBM bulk ground truth segmentation tool: "was segmented using DNAcopy (v1.62.0)47 (Supplementary Data 5)."
- [discussion] MM integer copy number cutoffs: "The integer Copy Number was assigned based on cutoffs specified in the CNVkit documentation (−1, −0.25, 0.2, and 0.7) (Supplementary Data 4)."
- [methods] Concrete SCEVAN invocation: "results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)"
- [readme] Repository usage of pipelineCNA: [evidence span withheld — anchor status: external_doc]
