---
name: multi-platform-scrna-dataset-harmonization
description: Use when automatically classifying malignant vs non-malignant cells from
  raw scRNA-seq count matrices spanning different cancer types and sequencing platforms
  (Smart-seq2 vs 10X Chromium), and when benchmarking that classification against
  CopyKAT using per-sample F1 scores.
when_to_use_negative:
- The sample is expected to contain only malignant cells with minimal genomic alterations
  (e.g. leukemia, pediatric cancers, Ependymomas) — aneuploidy-based classification
  is unsuitable there
- You already have manually identified reference normal-cell clusters and want a supervised
  inferCNV-style analysis rather than fully automatic end-to-end classification
- The goal is clonal substructure and phylogeny rather than pure malignant/non-malignant
  classification (then set SUBCLONES = TRUE, outside this skill's scope)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: SCEVAN
  role: Runs the automatic malignant/non-malignant classification across all 106 samples
    via pipelineCNA with automatic confident-normal-cell detection (yaGST, P < 1e-10,
    NES > 1.0)
  repo: https://github.com/AntonioDeFalco/SCEVAN
- name: CopyKAT v1.0.5
  role: Baseline automatic malignant/non-malignant classifier run with default workflow
    on the same count matrices for per-sample F1 comparison
- name: yaGST
  role: Mann–Whitney–Wilcoxon single-sample gene set test used inside SCEVAN to identify
    high-confidence normal cells (P < 1e-10, NES > 1.0)
  repo: https://github.com/AntonioDeFalco/SCEVAN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:47d9b89b4e8a7471bf9bf51089bdff8318cda1c1cd2e9059b92c0daf06d2b01f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AntonioDeFalco/SCEVAN
  tools:
  - CopyKAT v1.0.5
  - SCEVAN
  - yaGST
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# multi-platform-scrna-dataset-harmonization

## Summary

Run SCEVAN's fully automatic malignant/non-malignant cell classification across 106 real scRNA-seq samples from multiple cancer types (GBM, HNSCC, colorectal) and platforms (Smart-seq2 and 10X Chromium), benchmarking per-sample F1 against CopyKAT v1.0.5 (SCEVAN 0.90 vs CopyKAT 0.63, winning in 63% of samples).

## When to use

Use when automatically classifying malignant vs non-malignant cells from raw scRNA-seq count matrices spanning different cancer types and sequencing platforms (Smart-seq2 vs 10X Chromium), and when benchmarking that classification against CopyKAT using per-sample F1 scores.

## When NOT to use

- The sample is expected to contain only malignant cells with minimal genomic alterations (e.g. leukemia, pediatric cancers, Ependymomas) — aneuploidy-based classification is unsuitable there
- You already have manually identified reference normal-cell clusters and want a supervised inferCNV-style analysis rather than fully automatic end-to-end classification
- The goal is clonal substructure and phylogeny rather than pure malignant/non-malignant classification (then set SUBCLONES = TRUE, outside this skill's scope)

## Inputs

- Raw gene-by-cell count matrices for each sample (gene symbols or Ensembl IDs on rows, cells on columns), e.g. GBM GSE131928 (MGH102/MGH104/MGH105/MGH106), HNSCC primary and lymph-node matrices (GSE10332), colorectal samples from Supplementary Data 2
- Platform label per sample (Smart-seq2 vs 10X Chromium)
- Reference copy number ground truth per sample derived from bulk tumor profiling (WES/WGS)

## Outputs

- Per-cell malignant/non-malignant classification from SCEVAN::pipelineCNA (SUBCLONES = FALSE)
- Per-cell malignant/non-malignant classification from CopyKAT v1.0.5
- Per-sample F1 scores for each method against bulk-derived copy number ground truth
- Aggregate F1 summary CSV (SCEVAN 0.90 vs CopyKAT 0.63, SCEVAN better in 63% of samples) and per-sample classification tables

## How to apply

Download the count matrices for each sample — e.g. GBM samples MGH102/MGH104/MGH105/MGH106 from GSE131928, HNSCC primary and lymph-node matrices (GSE10332), and colorectal samples from Supplementary Data 2 — retaining each sample's platform label (Smart-seq2 vs 10X Chromium). For each sample, run SCEVAN::pipelineCNA on the raw count matrix with par_cores = 20 and SUBCLONES = FALSE, so the tool's automatic confident-normal-cell detection (yaGST signature tests, P < 1e-10, NES > 1.0) drives a purely unsupervised malignant/non-malignant call per cell. Run CopyKAT v1.0.5 on the same count matrices with its default workflow to obtain the competing automatic classification. Compare both methods' per-cell labels against copy number ground truth derived from bulk data (resampling for the different resolution of WES/WGS) and compute per-sample F1 scores. Aggregate F1 across all samples, count the samples where SCEVAN beats CopyKAT, and write per-sample classification tables plus the aggregate F1 summary to CSV.

## Related tools

- **SCEVAN** (Runs the automatic malignant/non-malignant classification across all 106 samples via pipelineCNA with automatic confident-normal-cell detection (yaGST, P < 1e-10, NES > 1.0)) — https://github.com/AntonioDeFalco/SCEVAN
- **CopyKAT v1.0.5** (Baseline automatic malignant/non-malignant classifier run with default workflow on the same count matrices for per-sample F1 comparison)
- **yaGST** (Mann–Whitney–Wilcoxon single-sample gene set test used inside SCEVAN to identify high-confidence normal cells (P < 1e-10, NES > 1.0)) — https://github.com/AntonioDeFalco/SCEVAN

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = FALSE)
```

## Evaluation signals

- Aggregate F1 across the 106 samples matches the reported values within tolerance: SCEVAN 0.90 and CopyKAT 0.63
- SCEVAN outperforms CopyKAT in 63% of the samples (about 67 of 106 samples)
- Per-sample F1 scores are computed against bulk-derived copy number ground truth for every sample, with each sample's platform (Smart-seq2 vs 10X Chromium) retained in the per-sample tables
- SCEVAN runs fully unsupervised per sample: no manually supplied normal cells, with automatic confident-normal-cell detection active (yaGST P < 1e-10, NES > 1.0)
- Total cohort scale matches the benchmark: 106 samples and 93,322 cells across tumor types and technologies

## Limitations

- Aneuploidy-based assumption is unsuitable for tumors with minimal genomic alterations such as leukemia, pediatric cancers, and Ependymomas
- Ground truth extracted from WES/WGS has different resolution than single-cell data and must be resampled before comparison
- CopyKAT classification can be affected by wrong identification of normal cells and was not designed for complete automatic end-to-end clone identification; inferCNV-like methods additionally perform sub-optimally on droplet-based (10X Chromium) data with sparse coverage

## Evidence

- [readme] 106-sample F1 benchmark: [evidence span withheld — anchor status: external_doc]
- [readme] SUBCLONES=FALSE classification scope: [evidence span withheld — anchor status: not_found]
- [discussion] confident normal cell detection thresholds: "assume as normal confident cells the top classified cells with P value less than 10−10 and Normalized Enrichment Score (NES) greater than 1.0."
- [intro] droplet data limitation: "they exhibit sub-optimal performances on droplet-based methods with very sparse coverage depth and higher throughput10."
- [discussion] ground-truth resolution caveat: "The ground truth extracted from WES and WGS have of course different resolutions with respect to the single-cell data."
