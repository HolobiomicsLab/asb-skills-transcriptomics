---
name: multiregional-glioblastoma-scrna-processing
description: Use when processing multiregional GBM (or analogous tumor) scRNA-seq
  count matrices to infer clonal copy number substructure and benchmark inferred CNA
  profiles against bulk WGS/WES segmented ground truth.
when_to_use_negative:
- Tumors with minimal genomic alterations (e.g., leukemias, pediatric cancers, ependymomas)
  where aneuploidy-based inference is unsuitable
- Samples where only a pre-computed feature table (not raw counts) is available, since
  SCEVAN requires the raw count matrix
- Analyses requiring manually specified reference cell clusters rather than automatic
  confident-normal-cell detection
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3518
tools:
- name: SCEVAN
  role: 'Primary R package: pipelineCNA infers per-cell CN profiles, malignant/non-malignant
    classification, and subclones; multiSampleComparisonClonalCN compares clonal profiles
    across samples'
  repo: github.com/AntonioDeFalco/SCEVAN
- name: yaGST
  role: Mann-Whitney-Wilcoxon single-sample gene set test used to identify high-confidence
    normal cells (P < 1e-10, NES > 1.0)
  repo: github.com/miccec/yaGST
- name: CopyKAT
  role: 'Comparator: state-of-the-art automatic malignant/non-malignant classifier
    run on the same samples for benchmarking'
- name: inferCNV
  role: 'Comparator: CNA inference benchmarked against SCEVAN on the same samples'
- name: DNAcopy
  role: Segments low-depth bulk WGS ground truth (v1.62.0) for GBM cohort, resampled
    to 1-Mb bins for correlation
- name: CNVkit
  role: Segments bulk WES ground truth (v0.9.9) with integer cutoffs -1, -0.25, 0.2,
    0.7 for MM cohort
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:d1d2988fe7d28fce02c63a10372536135cfcd1e083eb46f8afc470afe00f930e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CNVkit
  - CopyKAT
  - DNAcopy
  - SCEVAN
  - inferCNV
  - yaGST
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# multiregional-glioblastoma-scrna-processing

## Summary

A reproducible workflow for inferring per-cell copy number profiles and malignant/non-malignant classifications from multiregional glioblastoma scRNA-seq samples (e.g., MGH102/MGH104/MGH105/MGH106 from GSE131928) using SCEVAN, and validating them against 1-Mb resampled bulk WGS/WES ground truth via Pearson correlation.

## When to use

Use when processing multiregional GBM (or analogous tumor) scRNA-seq count matrices to infer clonal copy number substructure and benchmark inferred CNA profiles against bulk WGS/WES segmented ground truth.

## When NOT to use

- Tumors with minimal genomic alterations (e.g., leukemias, pediatric cancers, ependymomas) where aneuploidy-based inference is unsuitable
- Samples where only a pre-computed feature table (not raw counts) is available, since SCEVAN requires the raw count matrix
- Analyses requiring manually specified reference cell clusters rather than automatic confident-normal-cell detection

## Inputs

- Raw scRNA-seq count matrix per sample (genes x cells, gene symbols or Ensembl IDs), e.g., MGH106.RData / listCountMtx.RData from GSE131928
- Bulk WGS ground truth segmented with DNAcopy v1.62.0 (Supplementary Data 5) for GBM samples
- Bulk WES ground truth segmented with CNVkit v0.9.9, integer cutoffs -1, -0.25, 0.2, 0.7 (Supplementary Data 4) for MM samples
- Optional named list of count matrices for multi-sample comparison (multiSampleComparisonClonalCN)

## Outputs

- Per-cell malignant/non-malignant classification and subclone assignments (results data.frame from pipelineCNA)
- Inferred per-cell copy number matrix (CNAmtx) and segmentation file (.seg) per sample
- Phylogenetic tree of subclones (plotTree = TRUE)
- Per-sample Pearson correlations of inferred LogRatio vs 1-Mb resampled bulk truth; aggregated mean/max per tool (e.g., benchmark_correlations.csv)

## How to apply

Load raw scRNA count matrices (genes on rows, cells on columns; Gene Symbol or Ensembl ID accepted) for each of the multiregional samples and run SCEVAN::pipelineCNA per sample with SUBCLONES = TRUE and par_cores = 20, which automatically filters cells with <200 detected genes and genes expressed in <1% of cells, filters REACTOME cell-cycle genes, identifies confident normal cells via yaGST gene signatures (P < 1e-10, NES > 1.0), subtracts the normal-cell baseline, and performs greedy Mumford-Shah multichannel segmentation (beta_vega default 0.5). Resample the bulk ground truth to 1-Mb bins to match single-cell resolution: GBM truth from low-depth WGS segmented with DNAcopy v1.62.0 (Supplementary Data 5) and, for MM, WES segmented with CNVkit v0.9.9 using integer cutoffs -1, -0.25, 0.2, 0.7 (Supplementary Data 4). Compute Pearson correlations between SCEVAN's inferred LogRatio (segment mean) profile and the resampled bulk truth per sample; aggregate mean and max correlations across the GBM cohort (reported: mean 0.57, max 0.81 for GBM WGS; mean 0.44, max 0.71 for MM WES). Optionally compare against CopyKAT v1.0.5 and inferCNV v1.4.0 run on the same samples following the SCEVAN Workflow steps, and integrate the SCEVAN aneuploidy/subclone annotations into a Seurat object via AddMetaData.

## Related tools

- **SCEVAN** (Primary R package: pipelineCNA infers per-cell CN profiles, malignant/non-malignant classification, and subclones; multiSampleComparisonClonalCN compares clonal profiles across samples) — github.com/AntonioDeFalco/SCEVAN
- **yaGST** (Mann-Whitney-Wilcoxon single-sample gene set test used to identify high-confidence normal cells (P < 1e-10, NES > 1.0)) — github.com/miccec/yaGST
- **CopyKAT** (Comparator: state-of-the-art automatic malignant/non-malignant classifier run on the same samples for benchmarking)
- **inferCNV** (Comparator: CNA inference benchmarked against SCEVAN on the same samples)
- **DNAcopy** (Segments low-depth bulk WGS ground truth (v1.62.0) for GBM cohort, resampled to 1-Mb bins for correlation)
- **CNVkit** (Segments bulk WES ground truth (v0.9.9) with integer cutoffs -1, -0.25, 0.2, 0.7 for MM cohort)

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- Per-sample Pearson correlation of SCEVAN LogRatio vs 1-Mb resampled bulk WGS truth should be comparable to reported GBM values (mean 0.57, max 0.81)
- MM cohort WES correlations should be comparable to reported values (mean 0.44, max 0.71)
- Classification F1 across samples should approximate the reported 0.90 versus 0.63 for state-of-the-art tools
- Output files (annot .RData, CNAmtx.RData, .seg files) exist per sample and subclone labels show shared and specific alterations per clone
- Quality filters applied: cells with <200 detected genes and genes expressed in <1% of cells removed; REACTOME cell-cycle genes filtered before segmentation

## Limitations

- Aneuploidy-based approaches (including SCEVAN) may not suit tumors with minimal genomic alterations such as leukemias, pediatric cancers, and ependymomas
- Ground truth from WES/WGS has different resolution than single-cell data, so resampling to 1-Mb bins is required before correlation, introducing approximation
- inferCNV-like methods require manual identification of reference cell clusters and perform sub-optimally on sparse droplet-based data; CopyKAT classification can suffer from wrong identification of normal cells
- Version of SCEVAN itself and the beta segmentation constant value are not explicitly stated in the text

## Evidence

- [results] SCEVAN LogRatio Pearson correlations: [evidence span withheld — anchor status: not_found]
- [methods] pipelineCNA invocation: "results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)"
- [discussion] quality filtering: "the cells with less than 200 detected genes and the genes expressed in less than 1% of cells are removed."
- [discussion] confident normal cells: "assume as normal confident cells the top classified cells with P value less than 10−10 and Normalized Enrichment Score (NES) greater than 1.0."
- [discussion] ground truth resolution: "The ground truth extracted from WES and WGS have of course different resolutions with respect to the single-cell data."
- [readme] README usage: [evidence span withheld — anchor status: external_doc]
