---
name: cnv-heatmap-and-oncoprint-interpretation
description: Use when SCEVAN (pipelineCNA with SUBCLONES=TRUE) has produced heatmap/clone-tree/OncoPrint
  outputs for a scRNA-seq tumor sample and you must interpret truncal vs. shared vs.
  subclone-specific CNAs and verify them against published profiles.
when_to_use_negative:
- The tumor has minimal genomic alterations (e.g., leukemia, pediatric cancers, ependymomas)
  where the aneuploidy-based heatmap/OncoPrint view is uninformative per the article's
  limitation statement
- Only malignant vs. non-malignant classification is needed, with no interest in clonal
  structure — run pipelineCNA with SUBCLONES = FALSE instead of interpreting subclone
  plots
- The input is bulk WES/WGS segmentation data rather than a scRNA-seq count matrix;
  use bulk tools (CNVkit, DNAcopy) instead
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCEVAN
  role: Generates the heatmap, subclone-heatmap, clone-tree, consensus, and OncoHeat2
    figures via pipelineCNA(SUBCLONES=TRUE, plotTree=TRUE); plotCNA_withAnnotCells
    adds per-cell metadata tracks to the CNA heatmap
  repo: github.com/AntonioDeFalco/SCEVAN
- name: yaGST
  role: Mann–Whitney–Wilcoxon single-sample gene set test used for differential gene-set
    enrichment of subclone-specific alteration regions (e.g., UBE2T up-regulation
    in subclone 3)
  repo: github.com/miccec/yaGST
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:ed56649e3957f7b4039875fb4be4c0c3ad1c35783b8655fef7149d98c152fe42
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCEVAN
  - yaGST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# cnv-heatmap-and-oncoprint-interpretation

## Summary

Reading and validating the visualization outputs of SCEVAN's clonal copy-number pipeline — CNA heatmaps, subclone heatmaps, phylogenetic clone trees, consensus plots, and OncoHeat2/OncoPrint-style figures — to confirm truncal vs. subclonal alterations and link alteration regions to differential gene expression (e.g., UBE2T up-regulation).

## When to use

Use when SCEVAN (pipelineCNA with SUBCLONES=TRUE) has produced heatmap/clone-tree/OncoPrint outputs for a scRNA-seq tumor sample and you must interpret truncal vs. shared vs. subclone-specific CNAs and verify them against published profiles.

## When NOT to use

- The tumor has minimal genomic alterations (e.g., leukemia, pediatric cancers, ependymomas) where the aneuploidy-based heatmap/OncoPrint view is uninformative per the article's limitation statement
- Only malignant vs. non-malignant classification is needed, with no interest in clonal structure — run pipelineCNA with SUBCLONES = FALSE instead of interpreting subclone plots
- The input is bulk WES/WGS segmentation data rather than a scRNA-seq count matrix; use bulk tools (CNVkit, DNAcopy) instead

## Inputs

- Raw scRNA-seq count matrix (genes on rows, cells on columns; Gene Symbol or Ensembl ID)
- SCEVAN output folder figures: <sample>heatmap.png, <sample>heatmap_subclones.png, <sample>CloneTree.png, <sample>consensus.png, <sample>OncoHeat2.png
- Per-region DE volcano plots (e.g., <sample>-DEchr1-155320896-160358952_subclones.png)
- SCEVAN .seg file and CNAmtx_relat matrix (annot.RData / CNAmtx.RData) for region-level extraction

## Outputs

- Interpreted alteration profile: truncal, shared, and subclone-specific CNA calls per clone
- Verified clone-tree/OncoPrint correspondence (which alterations are truncal vs. subclonal)
- Region-level biological annotation linking amplified/deleted intervals to up/down-regulated genes (e.g., UBE2T at Chr 15 q31.2 region)

## How to apply

Run SCEVAN::pipelineCNA on the raw count matrix with SUBCLONES = TRUE and plotTree = TRUE so the pipeline emits heatmap, subclone-heatmap, clone-tree, consensus, and OncoHeat2 figures. Read the heatmap to confirm that multichannel segmentation kept cells in a copy number clone aligned at shared breakpoints; read the heatmap_subclones panel to separate truncal/shared events (e.g., Chr5 q23.2–q31.3 amplification, Chr10 q22.1–q26.3 deletion spanning PTEN) from subclone-specific events (e.g., 1q31.2 and 19q13.32 amplifications). Cross-check the OncoHeat2/OncoPrint plot against the clone tree so each alteration is assigned to exactly one clone level (truncal, shared, or specific to a numbered subclone). For any altered region, extract the corresponding DE volcano plot or use the .seg coordinates against the CNAmtx_relat matrix (as shown in the README's plotCNA_withAnnotCells workflow) to confirm driver genes in the region, e.g., UBE2T significantly up-regulated in subclone 3 (P = 2.69e-43, LFC = 1.10).

## Related tools

- **SCEVAN** (Generates the heatmap, subclone-heatmap, clone-tree, consensus, and OncoHeat2 figures via pipelineCNA(SUBCLONES=TRUE, plotTree=TRUE); plotCNA_withAnnotCells adds per-cell metadata tracks to the CNA heatmap) — github.com/AntonioDeFalco/SCEVAN
- **yaGST** (Mann–Whitney–Wilcoxon single-sample gene set test used for differential gene-set enrichment of subclone-specific alteration regions (e.g., UBE2T up-regulation in subclone 3)) — github.com/miccec/yaGST

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "BT1160", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- The classification data frame is produced and exactly three malignant subclones are inferred for the BT1160 sample
- The OncoPrint/OncoHeat2 plot shows the four reported alteration assignments matching the paper: truncal Chr5 q23.2–q31.3 amplification, shared Chr10 q22.1–q26.3 deletion, and subclone-specific 1q31.2 and 19q13.32 amplifications
- Heatmap shows cells within a clone sharing the same breakpoints (multichannel segmentation invariant)
- DE result for the altered region reproduces UBE2T significance in subclone 3 (P = 2.69e-43, LFC = 1.10)
- Clone tree topology is consistent with the heatmap: alterations on the trunk appear in all subclones, subclone-specific alterations appear in exactly one branch

## Limitations

- Aneuploidy-based interpretation is unsuitable for tumors with minimal genomic alterations (leukemia, pediatric cancers, ependymomas)
- CopyKAT-style normal-cell misclassification can distort the heatmap baseline; SCEVAN mitigates this with yaGST confident-normal-cell selection (P < 1e-10, NES > 1.0, max 30 cells)
- Segments within a clone representing the same CNAs are merged only when within 10 Mb, so very distant same-arm events may appear as separate heatmap blocks
- beta segmentation parameter (default 0.5) controls granularity: higher beta gives coarser-grained segments and can miss narrow alterations

## Evidence

- [readme] The plotCNA_withAnnotCells function generates a heatmap of the copy number profile for each cell, incorporating cell annotations as tracks.: [evidence span withheld — anchor status: external_doc]
- [abstract] It uses a multichannel segmentation algorithm exploiting shared breakpoints within copy number clones: "It uses a multichannel segmentation algorithm exploiting the assumption that all the cells in a given copy number clone share the same breakpoints"
- [other] SCEVAN identifies several truncal alterations, such as the amplification on Chr 5 (q23.2–q31.3), shared alterations, such as the deletion on Chr 10 (q22.1–q26.3).: [evidence span withheld — anchor status: not_found]
- [discussion] Segments in each clone representing the same copy number alterations at genomic distances less 10Mb are first merged together.: "Segments in each clone representing the same copy number alterations at genomic distances less 10Mb are first merged together."
