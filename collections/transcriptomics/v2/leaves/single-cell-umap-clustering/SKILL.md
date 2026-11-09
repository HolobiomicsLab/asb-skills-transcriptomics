---
name: single-cell-umap-clustering
description: Use when you have a per-cell TE count matrix (from scTE or scTEATAC on
  scRNA-seq/scATAC-seq BAMs) and need to verify that cells cluster by known identity
  using TE features alone.
when_to_use_negative:
- Input is already a gene-expression or peak-level feature table and gene-based cell-type
  clustering suffices — TE information content is lower than genes and serves mainly
  as an addendum.
- The goal is resolving individual enhancer peaks per cell in sparse scATAC-seq —
  merging TE data as enhancer information remains speculative and was not demonstrated.
- Bulk (non-single-cell) data, where per-cell barcodes/UMIs are absent and scTE's
  cell-by-TE quantification cannot be built.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3308
tools:
- name: scTEATAC
  role: Processes scATAC-seq BAM files against the TE index to produce per-cell TE
    accessibility counts used as clustering features
  repo: https://github.com/JiekaiLab/scTE
- name: scTEATAC_build
  role: Builds the mm10 TE index from the filtered RepeatMasker BED used by scTEATAC
  repo: https://github.com/JiekaiLab/scTE
- name: scTE
  role: Single-cell TE processing pipeline underlying TE quantification from single-cell
    sequencing data
  repo: https://github.com/JiekaiLab/scTE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21808-x
    title: Identifying transposable element expression dynamics and heterogeneity
      during development at the single-cell level with a processing pipeline scTE
  source_content_hash: sha256:6d9932d551e97ba86d75bf4a2b50e5f3e37211587a4c82f8888ec01bf5650c9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiekaiLab/scTE
  tools:
  - scTE
  - scTEATAC
  - scTEATAC_build
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# single-cell-umap-clustering

## Summary

Dimensionality reduction and UMAP clustering applied to a single-cell-by-TE accessibility matrix (.h5ad) built from scATAC-seq BAMs with scTE/scTEATAC, to test whether TE-mapped reads alone recover FACS-purified cell identities.

## When to use

Use when you have a per-cell TE count matrix (from scTE or scTEATAC on scRNA-seq/scATAC-seq BAMs) and need to verify that cells cluster by known identity using TE features alone.

## When NOT to use

- Input is already a gene-expression or peak-level feature table and gene-based cell-type clustering suffices — TE information content is lower than genes and serves mainly as an addendum.
- The goal is resolving individual enhancer peaks per cell in sparse scATAC-seq — merging TE data as enhancer information remains speculative and was not demonstrated.
- Bulk (non-single-cell) data, where per-cell barcodes/UMIs are absent and scTE's cell-by-TE quantification cannot be built.

## Inputs

- BAM files of FACS-purified scATAC-seq samples (mouse CPC, CD4+ T cells, ESCs, skin fibroblasts; human PBMC)
- mm10 RepeatMasker (rmsk) annotation from UCSC goldenPath, filtered to LINE/SINE/LTR/Retroposon classes (mm10.te.bed)
- scTEATAC index (mm10.te.atac.idx)
- Per-cell TE accessibility count matrix assembled into .h5ad

## Outputs

- Single-cell-by-TE accessibility matrix (.h5ad)
- UMAP embedding with cells separating by known FACS-purified identities
- Per-TE accessibility profiles confirming marker TE specificity (RLTR13A, RLTR4_Mm, RLTR13G, RMER19B/C)

## How to apply

After running scTEATAC on each FACS-purified input BAM (e.g. 'scTEATAC -i input.bam -x mm10.te.atac.idx'), assemble per-cell TE accessibility counts for mouse CPC, CD4+ T cell, ESC and skin fibroblast samples (plus human PBMC) into a single-cell-by-TE matrix and export it as .h5ad. Build the index from UCSC mm10 RepeatMasker annotation filtered with grep -E 'LINE|SINE|LTR|Retroposon' and cut -f6-8,11 into mm10.te.bed, then scTEATAC_build -g mm10.te.bed -o mm10.te.atac. Perform dimensionality reduction followed by UMAP embedding on TE accessibility alone — deliberately excluding gene features — and color/inspect cells by their known FACS-purified identities. Finally, check marker TEs (RLTR13A in CPCs, RLTR4_Mm in CD4+ T cells, RLTR13G in ESCs, RMER19B/C in skin fibroblasts) for cell-type-specific openness. The rationale is that TEs carry cis-regulatory sequences, so their merged accessibility can act as a low-dimensional cell-identity signal even where scATAC-seq per-peak data is too sparse.

## Related tools

- **scTEATAC** (Processes scATAC-seq BAM files against the TE index to produce per-cell TE accessibility counts used as clustering features) — https://github.com/JiekaiLab/scTE
- **scTEATAC_build** (Builds the mm10 TE index from the filtered RepeatMasker BED used by scTEATAC) — https://github.com/JiekaiLab/scTE
- **scTE** (Single-cell TE processing pipeline underlying TE quantification from single-cell sequencing data) — https://github.com/JiekaiLab/scTE

## Examples

```
scTEATAC -i input.bam -x mm10.te.atac.idx
```

## Evaluation signals

- UMAP clustering on TE accessibility alone must recover the expected FACS-purified cell-type groupings (CPC, CD4+ T cell, ESC, skin fibroblast; human PBMC)
- RLTR13A shows elevated accessibility specifically in CPCs
- RLTR4_MM shows elevated accessibility specifically in CD4+ T cells
- RLTR13G shows elevated accessibility specifically in ESCs
- RMER19B/C shows elevated accessibility specifically in skin fibroblasts

## Limitations

- TE information content is lower than that of genes; TEs are 'a useful addendum to the gene information' rather than a replacement.
- Using merged TE data as enhancer information in single-cell ATAC-seq remains speculative — individual peaks in individual cells are challenging to resolve due to scATAC-seq sparsity.
- Read allocation is priority-based: TEs inside exon/UTR regions of annotated genes contribute to the gene, not the TE score, unless --mode/-m inclusive is set; use nointron to exclude intronic TEs.

## Evidence

- [results] scTE applied to FACS-purified mouse scATAC-seq data accurately recovered expected cell types using TE-mapped reads alone: "Finally, we apply scTE to single-cell ATAC-seq data, and demonstrate that scTE can discriminate cell type using chromatin accessibly of TEs alone."
- [methods] Filter RepeatMasker annotation to LINE/SINE/LTR/Retroposon classes for the ATAC index: "$ zcat mm10.te.txt.gz | grep -E 'LINE|SINE|LTR|Retroposon' | cut -f6-8,11 >mm10.te.bed"
- [methods] Run scTEATAC on scATAC-seq BAMs to obtain per-cell TE accessibility: "scTEATAC -i input.bam -x mm10.te.atac.idx"
- [intro] TEs as cis-regulatory elements underpin the accessibility-based discrimination: "TEs carry cis-regulatory sequences and their duplication and insertion can reshape gene regulatory networks by redistributing transcription factor (TF) binding sites and evolving new enhancer"
- [discussion] TE features are subordinate to gene features in information content: "Overall, whilst the information content of TEs is lower than that of genes, TEs are a useful addendum to the gene information"
