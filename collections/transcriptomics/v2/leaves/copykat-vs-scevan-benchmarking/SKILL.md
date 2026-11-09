---
name: copykat-vs-scevan-benchmarking
description: Use when comparing automatic aneuploidy-based malignant cell callers
  (SCEVAN vs CopyKAT) on real scRNA-seq count matrices from GBM, HNSCC or colorectal
  tumors across Smart-seq2 and 10X Chromium platforms, with per-sample F1 aggregation.
when_to_use_negative:
- The goal is clonal substructure or phylogeny reconstruction rather than pure malignant/non-malignant
  classification (in that case SUBCLONES = TRUE is the appropriate scope, not the
  classification benchmark)
- The tumor type has minimal genomic alterations (e.g. leukemia, pediatric cancers,
  ependymomas), where aneuploidy-based classification is unsuited per the article's
  stated limitation
- The ground truth must come from single-cell DNA data rather than bulk WES/WGS profiles,
  which have different resolution and require resampling before comparison
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: SCEVAN
  role: Automatic malignant/non-malignant classifier under test; run via pipelineCNA
    with par_cores = 20, SUBCLONES = FALSE, using confident-normal-cell detection
    (yaGST, P < 1e-10, NES > 1.0)
  repo: https://github.com/AntonioDeFalco/SCEVAN
- name: CopyKAT v1.0.5
  role: State-of-the-art automatic baseline classifier run on the same count matrices
    with default workflow for comparison
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
  source_content_hash: sha256:a8402e223864eb47542a2dd96b3bfcab0e87835100a2a80b236b9fa0858494c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AntonioDeFalco/SCEVAN
  tools:
  - CopyKAT v1.0.5
  - SCEVAN
  - SCEVAN::pipelineCNA
  - yaGST
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# copykat-vs-scevan-benchmarking

## Summary

Benchmark of fully automatic malignant/non-malignant cell classification on real scRNA-seq data: SCEVAN's pipelineCNA (SUBCLONES = FALSE) versus CopyKAT v1.0.5, scored by per-sample F1 against bulk-derived copy number ground truth across 106 samples (93,322 cells) from GBM, HNSCC and colorectal tumors.

## When to use

Use when comparing automatic aneuploidy-based malignant cell callers (SCEVAN vs CopyKAT) on real scRNA-seq count matrices from GBM, HNSCC or colorectal tumors across Smart-seq2 and 10X Chromium platforms, with per-sample F1 aggregation.

## When NOT to use

- The goal is clonal substructure or phylogeny reconstruction rather than pure malignant/non-malignant classification (in that case SUBCLONES = TRUE is the appropriate scope, not the classification benchmark)
- The tumor type has minimal genomic alterations (e.g. leukemia, pediatric cancers, ependymomas), where aneuploidy-based classification is unsuited per the article's stated limitation
- The ground truth must come from single-cell DNA data rather than bulk WES/WGS profiles, which have different resolution and require resampling before comparison

## Inputs

- Raw scRNA-seq count matrices (genes x cells) for 106 real samples: GBM (GSE131928, e.g. MGH106), HNSCC primary and lymph-node samples, and colorectal samples from Supplementary Data 2
- Platform label per sample (Smart-seq2 vs 10X Chromium)
- Reference copy number ground truth per sample derived from bulk tumor profiling (low-depth WGS/WES)

## Outputs

- Per-sample malignant/non-malignant cell label vector from SCEVAN::pipelineCNA (SUBCLONES = FALSE)
- Per-sample malignant/non-malignant cell label vector from CopyKAT v1.0.5
- Per-sample F1 score table for both methods (CSV)
- Aggregate F1 summary (SCEVAN 0.90 vs CopyKAT 0.63; SCEVAN better in 63% of samples)

## How to apply

Download raw count matrices for GBM samples from GSE131928 (e.g. MGH106), HNSCC primary and lymph-node matrices, and colorectal samples listed in Supplementary Data 2, retaining each sample's platform label (Smart-seq2 vs 10X Chromium). Run SCEVAN::pipelineCNA with par_cores = 20 and SUBCLONES = FALSE so the tool performs pure malignant/non-malignant classification via its automatic confident-normal-cell detection (yaGST signature tests at P < 1e-10 and NES > 1.0). Run CopyKAT v1.0.5 on the same count matrices with its default workflow. Compare each method's predicted labels to bulk-derived copy number ground truth per sample, compute per-sample F1 scores, and aggregate: reported result is SCEVAN overall F1 = 0.90 vs CopyKAT = 0.63, with SCEVAN better in 63% of samples. Validation requires the reproduced aggregate F1 values to match these within tolerance and SCEVAN to win in 63% of samples; write per-sample tables and the aggregate summary to CSV.

## Related tools

- **SCEVAN** (Automatic malignant/non-malignant classifier under test; run via pipelineCNA with par_cores = 20, SUBCLONES = FALSE, using confident-normal-cell detection (yaGST, P < 1e-10, NES > 1.0)) — https://github.com/AntonioDeFalco/SCEVAN
- **CopyKAT v1.0.5** (State-of-the-art automatic baseline classifier run on the same count matrices with default workflow for comparison)
- **yaGST** (Mann–Whitney–Wilcoxon single-sample gene set test used inside SCEVAN to identify confident normal cells (P < 1e-10, NES > 1.0))

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = FALSE)
```

## Evaluation signals

- Reproduced aggregate F1 must equal 0.90 for SCEVAN and 0.63 for CopyKAT within tolerance
- SCEVAN must outperform CopyKAT in 63% of the 106 samples
- Per-sample classification must be run with SUBCLONES = FALSE (pure classification scope), not the subclone workflow
- Both methods must be run on identical raw count matrices, including Smart-seq2 and 10X Chromium samples, with platform labels retained
- Comparison must use bulk-derived copy number ground truth, resampled to single-cell resolution where needed

## Limitations

- Aneuploidy-based methods (including SCEVAN and CopyKAT) may not suit tumors with minimal genomic alterations such as leukemia, pediatric cancers and ependymomas
- Ground truth from WES/WGS has different resolution than single-cell data and requires resampling before F1 comparison
- CopyKAT classification can be affected by wrong identification of normal cells and was not designed for complete automatic end-to-end clone identification

## Evidence

- [abstract] Real-data benchmark headline numbers: "We apply SCEVAN to datasets encompassing 106 samples and 93,322 cells from different tumor types and technologies"
- [readme] F1 comparison: [evidence span withheld — anchor status: external_doc]
- [intro] Automatic classification rationale vs CopyKAT: "An approach to overcome these limitations is represented by the CopyKAT method11 that automatically classifies malignant and non-malignant cells."
- [methods] pipelineCNA invocation: "results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)"
- [discussion] Confident normal cell thresholds: "assume as normal confident cells the top classified cells with P value less than 10−10 and Normalized Enrichment Score (NES) greater than 1.0."
- [discussion] CopyKAT version used: "are analyzed following the steps of SCEVAN Workflow (Methods) and with CopyKAT v1.0.5 and inferCNV v1.4.0."
