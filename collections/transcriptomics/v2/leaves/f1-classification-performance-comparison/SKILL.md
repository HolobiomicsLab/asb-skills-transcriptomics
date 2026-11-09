---
name: f1-classification-performance-comparison
description: Use when comparing malignant-vs-non-malignant cell classifiers on scRNA-seq
  count matrices with bulk WES/WGS-derived copy number ground truth, and you need
  per-sample F1 aggregation across cancer types and platforms.
when_to_use_negative:
- The sample is believed to be pure tumor with minimal genomic alterations (e.g. leukemia,
  pediatric cancers, Ependymomas) — aneuploidy-based classifiers and their benchmarks
  are unsuitable there
- You only have synthetic/simulated data or no bulk-derived copy number reference
  — the F1 comparison requires reference ground truth from bulk tumor profiling
- Your goal is clonal substructure or phylogeny rather than pure malignant/non-malignant
  classification (use SUBCLONES = TRUE workflows instead)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: SCEVAN
  role: Automatic malignant/non-malignant classifier under test; run via pipelineCNA
    with SUBCLONES = FALSE, par_cores = 20, using automatic confident-normal-cell
    detection
  repo: github.com/AntonioDeFalco/SCEVAN
- name: CopyKAT v1.0.5
  role: Baseline automatic malignant/non-malignant classifier compared against SCEVAN
    using its default workflow
- name: yaGST
  role: Mann–Whitney–Wilcoxon single-sample gene set test used inside SCEVAN to identify
    confident normal cells (P < 1e-10, NES > 1.0)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:6b122207e8ac0ec60b02045efaad0bda656ad067b7fa4367fb8263e9c8b48baf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CopyKAT v1.0.5
  - SCEVAN
  - yaGST
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# f1-classification-performance-comparison

## Summary

Benchmark protocol for comparing automatic malignant/non-malignant cell classification of scRNA-seq methods (SCEVAN vs CopyKAT) by computing per-sample F1 scores against bulk-derived copy number ground truth across 106 real tumor samples.

## When to use

Use when comparing malignant-vs-non-malignant cell classifiers on scRNA-seq count matrices with bulk WES/WGS-derived copy number ground truth, and you need per-sample F1 aggregation across cancer types and platforms.

## When NOT to use

- The sample is believed to be pure tumor with minimal genomic alterations (e.g. leukemia, pediatric cancers, Ependymomas) — aneuploidy-based classifiers and their benchmarks are unsuitable there
- You only have synthetic/simulated data or no bulk-derived copy number reference — the F1 comparison requires reference ground truth from bulk tumor profiling
- Your goal is clonal substructure or phylogeny rather than pure malignant/non-malignant classification (use SUBCLONES = TRUE workflows instead)

## Inputs

- Raw scRNA-seq count matrices (genes on rows, cells on columns; gene symbols or Ensembl IDs) for each sample, e.g. GSE131928 GBM samples, HNSCC primary/lymph-node matrices, colorectal samples
- Platform label per sample (Smart-seq2 vs 10X Chromium)
- Bulk-derived copy number ground truth per sample (from low-depth WGS or WES, resampled to single-cell resolution before comparison)

## Outputs

- Per-sample predicted malignant/non-malignant cell labels from each method (SCEVAN classification table, CopyKAT classification)
- Per-sample F1 score for each method
- Aggregate F1 summary across all samples (e.g. SCEVAN 0.90 vs CopyKAT 0.63) and win-rate count (samples where SCEVAN outperforms CopyKAT), written to CSV

## How to apply

Download raw count matrices for each real scRNA-seq sample (e.g. GBM samples MGH102/MGH104/MGH105/MGH106 from GSE131928, HNSCC primary and lymph-node matrices, colorectal samples), retaining each sample's platform label (Smart-seq2 vs 10X Chromium). Run each classifier end-to-end in automatic mode: SCEVAN::pipelineCNA with par_cores = 20 and SUBCLONES = FALSE (so only the malignant/non-malignant call is evaluated), relying on its automatic confident-normal-cell detection (yaGST signature tests, P < 1e-10, NES > 1.0); run CopyKAT v1.0.5 with its default workflow. For each sample, compare predicted cell labels against the reference copy number ground truth derived from bulk data and compute the per-sample F1 score for each method. Aggregate F1 across all samples and count in which fraction of samples one method wins; in the reference benchmark (106 samples, 93,322 cells from GBM, HNSCC, colorectal) SCEVAN reaches overall F1 0.90 vs 0.63 for CopyKAT, outperforming it in 63% of samples.

## Related tools

- **SCEVAN** (Automatic malignant/non-malignant classifier under test; run via pipelineCNA with SUBCLONES = FALSE, par_cores = 20, using automatic confident-normal-cell detection) — github.com/AntonioDeFalco/SCEVAN
- **CopyKAT v1.0.5** (Baseline automatic malignant/non-malignant classifier compared against SCEVAN using its default workflow)
- **yaGST** (Mann–Whitney–Wilcoxon single-sample gene set test used inside SCEVAN to identify confident normal cells (P < 1e-10, NES > 1.0))

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = FALSE)
```

## Evaluation signals

- Reproduced aggregate F1 must match the reported values: SCEVAN 0.90 and CopyKAT 0.63 across the 106 samples, within tolerance
- SCEVAN must outperform CopyKAT in 63% of samples (count the winning samples across the per-sample F1 table)
- Per-sample F1 tables must contain a score for each of the 106 samples and both methods, with platform labels (Smart-seq2 vs 10X Chromium) retained
- SCEVAN's automatic normal-cell detection should yield confident normal cells with P < 1e-10 and NES > 1.0, or treat the sample as pure malignant if none are found
- CSV outputs should include per-sample classification tables and the aggregate F1 summary as specified in the workflow

## Limitations

- Aneuploidy-based classification may not be suited to tumors with minimal genomic alterations such as leukemia, pediatric cancers, and Ependymomas
- Ground truth extracted from WES and WGS has different resolution than single-cell data, requiring resampling before comparison
- CopyKAT classification can be affected by wrong identification of normal cells, which propagates into the baseline comparison
- inferCNV-like methods and this benchmark setup assume droplet-based data (very sparse coverage, higher throughput) is handled appropriately; such data degrades reference-cluster-based methods

## Evidence

- [readme] Overall F1 of 0.90 vs CopyKAT 0.63 across 106 samples: [evidence span withheld — anchor status: external_doc]
- [methods] Run SCEVAN pipelineCNA for classification only: "results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)"
- [discussion] Confident normal cell thresholds: "assume as normal confident cells the top classified cells with P value less than 10−10 and Normalized Enrichment Score (NES) greater than 1.0."
- [discussion] Pure-sample fallback: "If no confident normal cells are found, we assume that the sample is pure and contains only malignant cells."
- [intro] Automatic classification benchmarked against CopyKAT: "An approach to overcome these limitations is represented by the CopyKAT method11 that automatically classifies malignant and non-malignant cells."
- [discussion] Aneuploidy assumption limitation: "Thus, our approach (and similar) may not be suited in this case."
