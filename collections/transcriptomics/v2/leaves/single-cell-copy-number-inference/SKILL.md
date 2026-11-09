---
name: single-cell-copy-number-inference
description: Use when raw scRNA-seq count matrices from tumor biopsies (Smart-seq2
  or 10X Chromium) must be automatically segregated into malignant and non-malignant
  cells and subcloned by copy-number architecture without a manually identified reference-cell
  cluster.
when_to_use_negative:
- Tumors expected to have minimal genomic alterations (leukemias, pediatric cancers,
  ependymomas) where aneuploidy-based inference is unsuitable
- Samples where cell-type annotation already exists and only expression analysis is
  needed (input is already a classified/annotated feature table)
- Tasks requiring matched-tumor/normal bulk segmentation only (use DNAcopy or CNVkit
  on bulk data rather than single-cell inference)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCEVAN
  role: 'Primary tool: pipelineCNA performs quality filtering, confident-normal-cell
    detection, joint multichannel segmentation, malignant/non-malignant classification,
    and subclone/phylogeny inference in one call'
  repo: github.com/AntonioDeFalco/SCEVAN
- name: yaGST
  role: Mann-Whitney-Wilcoxon single-sample gene set test used to identify confident
    normal cells (P < 1e-10, NES > 1.0)
  repo: github.com/miccec/yaGST
- name: CopyKAT v1.0.5
  role: Baseline comparator for automatic malignant/non-malignant classification against
    which SCEVAN's F1 is benchmarked
- name: inferCNV v1.4.0
  role: Reference-based comparator whose inferred CNV profile is correlated against
    bulk ground truth
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:090bff0cd4dd2eb5517c7e065b61c508e6658f9d54fea1aa000975961b70702d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CopyKAT v1.0.5
  - SCEVAN
  - inferCNV
  - inferCNV v1.4.0
  - yaGST
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-copy-number-inference

## Summary

Infers per-cell malignant vs non-malignant labels and clonal copy-number substructure directly from raw scRNA-seq count matrices using SCEVAN's variational pipeline, without manually defined normal-cell references. Rationale: aneuploidy-driven expression profiles allow joint segmentation and classification that outperforms CopyKAT (F1 0.90 vs 0.63 across 106 samples).

## When to use

Use when raw scRNA-seq count matrices from tumor biopsies (Smart-seq2 or 10X Chromium) must be automatically segregated into malignant and non-malignant cells and subcloned by copy-number architecture without a manually identified reference-cell cluster.

## When NOT to use

- Tumors expected to have minimal genomic alterations (leukemias, pediatric cancers, ependymomas) where aneuploidy-based inference is unsuitable
- Samples where cell-type annotation already exists and only expression analysis is needed (input is already a classified/annotated feature table)
- Tasks requiring matched-tumor/normal bulk segmentation only (use DNAcopy or CNVkit on bulk data rather than single-cell inference)

## Inputs

- Raw scRNA-seq count matrix with genes on rows and cells on columns (gene symbols or Ensembl IDs), e.g. MGH106_data.RData from GSE131928
- Optional named list of raw count matrices for multi-sample clonal comparison (multiSampleComparisonClonalCN)
- Optional vector of known normal cells (norm_cells) with FIXED_NORMAL_CELLS flag
- Optional AdditionalGeneSets of normal cell-type signatures (e.g. Supplementary Data 1 stromal/immune signatures) and organism ('human' or 'mouse')

## Outputs

- Per-cell results data frame with tumor/normal classification, confident-normal-cell flags, and assigned clonal subpopulation (Seurat-ready metadata)
- Subclone-specific and shared CNA calls with consensus segment-level CNV states from the .seg / CNAmtx RData files
- Plot artifacts: sample heatmap PNGs (e.g. BT1160heatmap.png, BT1160heatmap_subclones.png, BT1160CloneTree.png, BT1160consensus.png, BT1160OncoHeat2.png)
- For benchmarks: per-sample F1 tables and Pearson correlations of inferred profiles vs 1-Mb-resampled bulk WGS/WES truth

## How to apply

Load the raw count matrix (genes on rows, gene symbol or Ensembl ID allowed) and run SCEVAN::pipelineCNA with par_cores = 20; set SUBCLONES = TRUE when clonal structure is needed and FALSE for pure malignant/non-malignant classification. The pipeline first removes cells with fewer than 200 detected genes and genes expressed in less than 1% of cells, annotates genes with Ensembl genomic locations and sorts them by coordinate, then filters REACTOME cell-cycle genes to reduce artificial segments. Confident normal cells are detected via yaGST signature tests (P < 1e-10, NES > 1.0, up to 30 cells); their median expression is subtracted as the copy-number baseline (if none are found, the sample is assumed pure and a synthetic Gaussian baseline is removed). Segmentation uses a greedy Mumford–Shah piecewise-constant minimization after edge-preserving TV smoothing with a logcosh prior (beta_vega, default 0.5, higher for more coarse-grained segmentation), exploiting shared breakpoints across cells of the same clone. Each cell is classified via a five-state truncated-normal mixture (deletion, loss, neutral, gain, amplification) fit by EM from fixed initial parameters (Supplementary Table 1) with per-segment majority vote; for multi-sample clonal comparison use multiSampleComparisonClonalCN on a named list of count matrices. Results integrate into Seurat via CreateSeuratObject/AddMetaData, and altered regions can be extracted from the .seg file and the CNA matrix as in the README chr3 example.

## Related tools

- **SCEVAN** (Primary tool: pipelineCNA performs quality filtering, confident-normal-cell detection, joint multichannel segmentation, malignant/non-malignant classification, and subclone/phylogeny inference in one call) — github.com/AntonioDeFalco/SCEVAN
- **yaGST** (Mann-Whitney-Wilcoxon single-sample gene set test used to identify confident normal cells (P < 1e-10, NES > 1.0)) — github.com/miccec/yaGST
- **CopyKAT v1.0.5** (Baseline comparator for automatic malignant/non-malignant classification against which SCEVAN's F1 is benchmarked)
- **inferCNV v1.4.0** (Reference-based comparator whose inferred CNV profile is correlated against bulk ground truth)

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- Aggregate F1 of malignant/non-malignant classification across the 106 real samples should reach ~0.90 (SCEVAN) vs 0.63 (CopyKAT), with SCEVAN winning in ~63% of samples
- Every segment in the consensus CNV table carries exactly one of the five state labels (deletion, loss, neutral, gain, amplification), consistent with the pipelineCNA classification data frame
- Pearson correlation of SCEVAN LogRatio profiles vs 1-Mb-resampled bulk truth should be ~0.57 mean (max 0.81) on GBM multiregional WGS and ~0.44 (max 0.71) on Multiple Myeloma WES
- On BT1160 the run should recover exactly three malignant subclones and the reported truncal/shared alterations (Chr5 q23.2-q31.3 amplification, Chr10 q22.1-q26.3 deletion/PTEN) plus UBE2T up-regulation in subclone 3 (P = 2.69e-43, LFC = 1.10)
- Returned results data frame must contain a per-cell classification and subclone column that can be added to a Seurat object via AddMetaData

## Limitations

- Aneuploidy-based assumption is unsuitable for tumors with minimal genomic alterations such as leukemia, pediatric cancers, and ependymomas
- If no confident normal cells are found, the sample is assumed pure and only malignant cells are inferred, risking misclassification when normals are rare but present
- Bulk WGS/WES ground truth has different resolution than single-cell data, requiring 1-Mb resampling before correlation comparison
- CopyKAT-style automatic classification can suffer from wrong identification of normal cells; inferCNV-like methods perform sub-optimally on sparse droplet-based data

## Evidence

- [abstract] SCEVAN can automatically and accurately discriminate between malignant and non-malignant cells, resulting in a practical framework to analyze tumors and their microenvironment: "SCEVAN can automatically and accurately discriminate between malignant and non-malignant cells, resulting in a practical framework"
- [abstract] It uses a multichannel segmentation algorithm exploiting the assumption that all the cells in a given copy number clone share the same breakpoints: "It uses a multichannel segmentation algorithm exploiting the assumption that all the cells in a given copy number clone share the same breakpoints"
- [discussion] Filtering and normal-cell detection thresholds: "the cells with less than 200 detected genes and the genes expressed in less than 1% of cells are removed."
- [discussion] Confident normal cell identification: "assume as normal confident cells the top classified cells with P value less than 10−10 and Normalized Enrichment Score (NES) greater than 1.0."
- [methods] Methods invocation: "results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)"
- [readme] README tool description: [evidence span withheld — anchor status: external_doc]
- [readme] README benchmark numbers: [evidence span withheld — anchor status: external_doc]
