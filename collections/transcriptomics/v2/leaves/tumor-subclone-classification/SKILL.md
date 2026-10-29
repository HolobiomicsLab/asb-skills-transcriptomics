---
name: tumor-subclone-classification
description: Use when starting from a raw scRNA-seq count matrix of a tumor sample
  and you need unsupervised malignant/non-malignant classification plus subclone-level
  copy-number substructure and clone phylogeny.
when_to_use_negative:
- Tumors with minimal genomic alterations (e.g. leukemias, pediatric cancers, ependymomas),
  since the aneuploidy-based approach may not be suited to these cases
- You only need to know whether a single region is amplified across cells and already
  have a Seurat object plus .seg/CNA matrices — plot the inferred CN ratio directly
  instead of rerunning the full subclone pipeline
- You need multi-sample clonal comparison — use multiSampleComparisonClonalCN on a
  named list of count matrices rather than single-sample pipelineCNA
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: SCEVAN
  role: Primary R package; pipelineCNA performs quality filtering, normal-cell identification,
    joint CNA segmentation, malignant/non-malignant classification, and subclone/phylogeny
    inference in a single run
  repo: github.com/AntonioDeFalco/SCEVAN
- name: yaGST
  role: Mann–Whitney–Wilcoxon single-sample gene set test used to identify confident
    normal cells and to test subclone-specific differential expression in alteration
    regions
  repo: github.com/miccec/yaGST
- name: CopyKAT
  role: Benchmark comparator for automatic malignant/non-malignant classification
    (v1.0.5); CopyKAT-style classification can mis-identify normal cells and was not
    designed for end-to-end clone identification
- name: inferCNV
  role: Benchmark comparator (v1.4.0); requires manual identification of reference
    cell clusters, which SCEVAN automates
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:d7088cd07cc4eb163fe9a96a9d0b1533f36a654861339e41edec2ec109cd658f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CopyKAT
  - SCEVAN
  - inferCNV
  - yaGST
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# tumor-subclone-classification

## Summary

Automatically classify malignant vs non-malignant cells from a raw scRNA-seq count matrix and resolve clonal copy-number substructure (truncal, shared, subclone-specific CNAs plus clone phylogeny) using SCEVAN's variational multichannel segmentation, as applied to GBM samples like BT1160 and MGH106.

## When to use

Use when starting from a raw scRNA-seq count matrix of a tumor sample and you need unsupervised malignant/non-malignant classification plus subclone-level copy-number substructure and clone phylogeny.

## When NOT to use

- Tumors with minimal genomic alterations (e.g. leukemias, pediatric cancers, ependymomas), since the aneuploidy-based approach may not be suited to these cases
- You only need to know whether a single region is amplified across cells and already have a Seurat object plus .seg/CNA matrices — plot the inferred CN ratio directly instead of rerunning the full subclone pipeline
- You need multi-sample clonal comparison — use multiSampleComparisonClonalCN on a named list of count matrices rather than single-sample pipelineCNA

## Inputs

- Raw scRNA-seq count matrix (genes on rows, cells on columns; Gene Symbol or Ensembl ID), e.g. BT1160 from the Neftel GBM dataset or MGH106 from GSE131928
- Optional: vector of known normal cell barcodes (norm_cells) and FIXED_NORMAL_CELLS flag
- Optional: AdditionalGeneSets of extra normal-cell signatures; organism ('human' or 'mouse')

## Outputs

- Per-cell results data frame with tumor/normal classification, confident-normal-cell flag, and subclone assignment
- Output plots: <sample>heatmap.png, <sample>heatmap_subclones.png, <sample>CloneTree.png, <sample>consensus.png, <sample>OncoHeat2.png
- Subclone-specific differential-expression volcano plots for alteration regions (e.g. BT1160-DEchr1-155320896-160358952_subclones.png)
- CNA matrices and .seg files (e.g. output/<sample>_CNAmtx.RData, <sample>_count_mtx_annot.RData) for region-level plotting in Seurat

## How to apply

Load the sample's raw count matrix (genes on rows, cells on columns; Gene Symbols or Ensembl IDs) into R and call SCEVAN::pipelineCNA(count_mtx, sample = '<sample>', par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE), which in one run filters low-quality cells (<200 detected genes) and genes expressed in <1% of cells, removes REACTOME cell cycle pathway genes, annotates genes with Ensembl genomic coordinates and sorts them, identifies confident normal cells via yaGST gene-signature testing (P < 1e-10, NES > 1.0, up to 30 cells), subtracts the normal-cell median baseline, and applies edge-preserving TV smoothing with a logcosh prior before greedy Mumford–Shah joint segmentation. If SUBCLONES = TRUE, it further merges same-CNA segments closer than 10 Mb, infers subclones, and plots the clone phylogeny; set beta_vega (default 0.5) higher for coarser segmentation. Read the returned per-cell data frame (tumor/normal classification, confident-normal flag, subclone assignment) and the output folder plots (heatmap, subclone heatmap, CloneTree, consensus, OncoHeat) to confirm truncal/shared/subclone-specific alterations such as the BT1160 Chr5q23.2–q31.3 amplification, Chr10q22.1–q26.3 deletion, and subclone-specific 1q31.2 and 19q13.32 amplifications. Optionally integrate per-cell classification and subclone labels into a Seurat object, and use yaGST differential testing of alteration regions (e.g. UBE2T up in subclone 3 of BT1160, P = 2.69e-43, LFC = 1.10) to validate subclone-specific biology.

## Related tools

- **SCEVAN** (Primary R package; pipelineCNA performs quality filtering, normal-cell identification, joint CNA segmentation, malignant/non-malignant classification, and subclone/phylogeny inference in a single run) — github.com/AntonioDeFalco/SCEVAN
- **yaGST** (Mann–Whitney–Wilcoxon single-sample gene set test used to identify confident normal cells and to test subclone-specific differential expression in alteration regions) — github.com/miccec/yaGST
- **CopyKAT** (Benchmark comparator for automatic malignant/non-malignant classification (v1.0.5); CopyKAT-style classification can mis-identify normal cells and was not designed for end-to-end clone identification)
- **inferCNV** (Benchmark comparator (v1.4.0); requires manual identification of reference cell clusters, which SCEVAN automates)

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "BT1160", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- A per-cell results data frame is returned where every cell has a tumor/normal classification, a confident-normal flag, and a subclone assignment
- With SUBCLONES = TRUE, expected subclones are recovered (e.g. exactly three malignant subclones for BT1160) with truncal, shared, and subclone-specific alterations matching the reference profile (BT1160: Chr5q23.2–q31.3 amplification, Chr10q22.1–q26.3 deletion, subclone-specific 1q31.2 and 19q13.32 amplifications)
- Output folder contains the expected files: <sample>heatmap.png, <sample>heatmap_subclones.png, <sample>CloneTree.png, <sample>consensus.png, <sample>OncoHeat2.png, and DE volcano plots for subclone-specific regions
- Differential expression in subclone-specific regions reproduces reported statistics (e.g. UBE2T up-regulated in BT1160 subclone 3, P = 2.69e-43, LFC = 1.10)
- Against ground truth from bulk WES/WGS profiling, classification F1 near the reported 0.90 (vs 0.63 for state-of-the-art comparators) is expected

## Limitations

- The aneuploidy-based assumption makes SCEVAN (and similar methods) unsuitable for tumors with minimal genomic alterations such as leukemia, pediatric cancers, and ependymomas
- If no confident normal cells are found, the sample is assumed pure and only a synthetic gaussian baseline is removed, which can affect classification in mixed samples
- Ground truth from WES/WGS has different resolution than single-cell data and must be resampled before comparison
- Only 30 top-classified cells are taken as confident normal cells (P < 1e-10, NES > 1.0), and beta_vega (default 0.5) controls segmentation granularity — higher beta gives more coarse-grained segmentation
- SCEVAN's own version number is not stated in the article, unlike compared tool versions (CopyKAT v1.0.5, inferCNV v1.4.0)

## Evidence

- [abstract] SCEVAN can automatically and accurately discriminate between malignant and non-malignant cells: "SCEVAN can automatically and accurately discriminate between malignant and non-malignant cells, resulting in a practical framework to analyze tumors and their microenvironment"
- [discussion] Filtering thresholds for cells and genes: "the cells with less than 200 detected genes and the genes expressed in less than 1% of cells are removed."
- [discussion] Confident normal cell criteria via yaGST: "assume as normal confident cells the top classified cells with P value less than 10−10 and Normalized Enrichment Score (NES) greater than 1.0."
- [methods] Core invocation pattern from methods: "results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)"
- [discussion] Segment merging rule for subclone CNA calls: "Segments in each clone representing the same copy number alterations at genomic distances less 10Mb are first merged together."
- [readme] README describes the package's end-to-end scope: [evidence span withheld — anchor status: external_doc]
