---
name: clonal-phylogeny-interpretation
description: Use when a single-sample scRNA-seq tumor count matrix must be resolved
  into malignant subclones with truncal vs shared vs subclone-specific copy-number
  alterations and a clone phylogeny, as in glioblastoma samples from GSE131928.
when_to_use_negative:
- Tumors with minimal genomic aneuploidy (e.g., leukemia, pediatric cancers, ependymomas),
  where aneuploidy-based clonal deconvolution is unsuited per the article's own limitation
  statement
- When only malignant vs non-malignant classification is needed and no clonal structure
  is of interest, SUBCLONES can be disabled rather than interpreting phylogeny outputs
- Input is already segmented copy-number data (e.g., bulk WGS segments from CNVkit/DNAcopy)
  rather than a raw single-cell count matrix
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3685
tools:
- name: SCEVAN
  role: Runs the end-to-end pipelineCNA that segments CNA jointly across cells, infers
    subclones, and plots the phylogenetic clone tree
  repo: github.com/AntonioDeFalco/SCEVAN
- name: yaGST
  role: Mann–Whitney–Wilcoxon single-sample gene set test used to identify confident
    normal cells and to test gene signatures supporting clone interpretation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:a8b36eeda7c4c2bf56c20fd22c302fb25fae3eddfd5ef073c03f3ac89a65a378
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCEVAN
  - yaGST
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3685
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# clonal-phylogeny-interpretation

## Summary

Interprets the clonal copy-number substructure and phylogeny inferred by SCEVAN from scRNA-seq data, distinguishing truncal, shared, and subclone-specific CNA events (e.g., in the BT1160 glioblastoma sample) and validating them against marker-gene differential expression such as UBE2T up-regulation in subclone 3.

## When to use

Use when a single-sample scRNA-seq tumor count matrix must be resolved into malignant subclones with truncal vs shared vs subclone-specific copy-number alterations and a clone phylogeny, as in glioblastoma samples from GSE131928.

## When NOT to use

- Tumors with minimal genomic aneuploidy (e.g., leukemia, pediatric cancers, ependymomas), where aneuploidy-based clonal deconvolution is unsuited per the article's own limitation statement
- When only malignant vs non-malignant classification is needed and no clonal structure is of interest, SUBCLONES can be disabled rather than interpreting phylogeny outputs
- Input is already segmented copy-number data (e.g., bulk WGS segments from CNVkit/DNAcopy) rather than a raw single-cell count matrix

## Inputs

- Raw scRNA-seq count matrix with genes on rows (Gene Symbol or Ensembl ID) and cells on columns (e.g., BT1160 or MGH106 from GSE131928)
- Sample name string used to label output plots and files
- Optional named list of count matrices for multi-sample clonal comparison (multiSampleComparisonClonalCN)

## Outputs

- Per-cell data frame with tumor/normal classification, confident-normal-cell flag, and assigned clonal subpopulation
- Output plots: sample heatmap, subclone heatmap, clone tree (sampleCloneTree.png), consensus plot, OncoHeat2 plot
- Subclone-specific differential-expression volcano plots over altered genomic regions (e.g., BT1160-DEchr1-155320896-160358952_subclones.png)
- .seg / CNA matrix files (e.g., sample_CNAmtx.RData, sample_count_mtx_annot.RData) for single-cell CN visualization

## How to apply

Load the raw scRNA-seq count matrix (genes on rows, cells on columns; Gene Symbol or Ensembl IDs accepted) and run a single SCEVAN::pipelineCNA call with sample name, par_cores, SUBCLONES = TRUE and plotTree = TRUE so quality filtering, confident-normal-cell identification via yaGST signatures (P < 1e-10, NES > 1.0), joint Mumford–Shah multichannel segmentation, and subclone/phylogeny inference run end-to-end. Inspect the returned per-cell data frame for tumor/normal classification and assigned clonal subpopulation, then read the emitted output files (heatmap, subclone heatmap, clone tree, consensus, OncoHeat plots) to assign each alteration as truncal, shared, or subclone-specific. Validate the interpretation by checking differential-expression volcano plots over subclone-specific regions (e.g., UBE2T up-regulated in subclone 3 with P = 2.69e-43, LFC = 1.10) and by confirming the number of subclones and alteration assignments match the reference profile (e.g., Chr5q truncal amplification, Chr10q/PTEN shared deletion, 1q31.2 and 19q13.32 subclone-specific amplifications).

## Related tools

- **SCEVAN** (Runs the end-to-end pipelineCNA that segments CNA jointly across cells, infers subclones, and plots the phylogenetic clone tree) — github.com/AntonioDeFalco/SCEVAN
- **yaGST** (Mann–Whitney–Wilcoxon single-sample gene set test used to identify confident normal cells and to test gene signatures supporting clone interpretation)

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "BT1160", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- The returned classification data frame assigns every cell as malignant/non-malignant and recovers exactly the expected number of malignant subclones (e.g., three for BT1160)
- Output folder contains all expected files: heatmap, heatmap_subclones, CloneTree, consensus, and OncoHeat2 plots
- OncoPrint-like OncoHeat plot reproduces the reported alteration assignments: truncal Chr5 q23.2–q31.3 amplification, shared Chr10 q22.1–q26.3 deletion, subclone-specific 1q31.2 and 19q13.32 amplifications
- Differential-expression test for a marker gene in the altered region reproduces reported statistics (UBE2T in subclone 3: P = 2.69e-43, LFC = 1.10)
- Confident normal cells, when found, satisfy P < 1e-10 and NES > 1.0 from the yaGST signature tests (max 30 cells)

## Limitations

- Aneuploidy-based approach may not be suited to tumors with minimal genomic alterations such as leukemia, pediatric cancers, and ependymomas
- If no confident normal cells are found, SCEVAN assumes a pure tumor sample and subtracts a synthetic gaussian baseline, which may misanchor clonal CN inference
- Segments in each clone representing the same CNAs are merged only at genomic distances less than 10 Mb, so distant same-clone events may be split; the beta segmentation parameter (default 0.5) controls coarseness of segmentation

## Evidence

- [results] SCEVAN identifies several truncal alterations, such as the amplification on Chr 5 (q23.2–q31.3), shared alterations, such as the deletion on Chr 10 (q22.1–q26.3).: [evidence span withheld — anchor status: not_found]
- [methods] pipelineCNA call with subclones and tree: "results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)"
- [discussion] Confident normal cell identification thresholds: "assume as normal confident cells the top classified cells with P value less than 10−10 and Normalized Enrichment Score (NES) greater than 1.0."
- [discussion] Segment merging rule: "Segments in each clone representing the same copy number alterations at genomic distances less 10Mb are first merged together."
- [discussion] Aneuploidy limitation: "our approach (and similar) may not be suited in this case."
- [readme] README usage parameters: [evidence span withheld — anchor status: not_found]
