---
name: scatac-seq-te-accessibility-quantification
description: Use when scATAC-seq BAM files (e.g. FACS-purified mouse cell samples)
  should be characterized by TE open-chromatin content alone, to test whether TE accessibility
  recovers known cell identities or reveals TE-specific openness.
when_to_use_negative:
- The input is single-cell RNA-seq data quantifying TE expression rather than chromatin
  accessibility — use scTE with scTE_build indices instead of scTEATAC.
- The analysis requires standard peak-based ATAC-seq calls or gene-level accessibility
  rather than TE-mapped reads.
- A genome assembly or RepeatMasker annotation for which no rmsk goldenPath download
  exists has not been prepared as a custom TE BED.
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3169
tools:
- name: scTEATAC
  role: Processes scATAC-seq BAM files against a TE index to produce per-cell TE accessibility
    counts
  repo: github.com/JiekaiLab/scTE
- name: scTEATAC_build
  role: Builds the scATAC-seq TE index from the filtered RepeatMasker TE BED file
  repo: github.com/JiekaiLab/scTE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21808-x
    title: Identifying transposable element expression dynamics and heterogeneity
      during development at the single-cell level with a processing pipeline scTE
  source_content_hash: sha256:997cc8061c8edc1bf2e7bd5377d9195270474d89d703dbb1276d4d6b3c737d68
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scTEATAC
  - scTEATAC_build
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# scatac-seq-te-accessibility-quantification

## Summary

Quantify per-cell chromatin accessibility of transposable elements from scATAC-seq BAM files using scTEATAC, then use the resulting cell-by-TE accessibility matrix alone to discriminate cell types and identify cell-type-specific open TEs.

## When to use

Use when scATAC-seq BAM files (e.g. FACS-purified mouse cell samples) should be characterized by TE open-chromatin content alone, to test whether TE accessibility recovers known cell identities or reveals TE-specific openness.

## When NOT to use

- The input is single-cell RNA-seq data quantifying TE expression rather than chromatin accessibility — use scTE with scTE_build indices instead of scTEATAC.
- The analysis requires standard peak-based ATAC-seq calls or gene-level accessibility rather than TE-mapped reads.
- A genome assembly or RepeatMasker annotation for which no rmsk goldenPath download exists has not been prepared as a custom TE BED.

## Inputs

- Aligned scATAC-seq BAM files (unfiltered alignment recommended, with cell barcodes)
- UCSC RepeatMasker rmsk.txt.gz annotation for mm10 (or other goldenPath assemblies)
- Filtered TE BED file restricted to LINE, SINE, LTR and Retroposon classes

## Outputs

- Per-cell TE accessibility count matrix from scTEATAC
- Single-cell-by-TE accessibility matrix (.h5ad)
- UMAP embedding showing cells separated by FACS-purified identities
- Accessibility profiles of specific marker TEs (RLTR13A, RLTR4_Mm, RLTR13G, RMER19B/C)

## How to apply

Download the mm10 RepeatMasker rmsk.txt.gz from UCSC goldenPath, filter with `grep -E 'LINE|SINE|LTR|Retroposon'`, cut columns 6-8 and 11 into mm10.te.bed, and build the ATAC index with `scTEATAC_build -g mm10.te.bed -o mm10.te.atac`. Process each scATAC-seq BAM with `scTEATAC -i input.bam -x mm10.te.atac.idx` to obtain per-cell TE accessibility counts. Assemble counts across samples (mouse CPCs, CD4+ T cells, ESCs, skin fibroblasts; human PBMCs) into a single-cell-by-TE matrix exported as .h5ad, then run dimensionality reduction and UMAP clustering using TE accessibility alone. Finally, inspect accessibility of candidate marker TEs (RLTR13A, RLTR4_Mm, RLTR13G, RMER19B/C) to confirm cell-type-specific openness.

## Related tools

- **scTEATAC** (Processes scATAC-seq BAM files against a TE index to produce per-cell TE accessibility counts) — github.com/JiekaiLab/scTE
- **scTEATAC_build** (Builds the scATAC-seq TE index from the filtered RepeatMasker TE BED file) — github.com/JiekaiLab/scTE

## Examples

```
scTEATAC_build -g mm10.te.bed -o mm10.te.atac && scTEATAC -i input.bam -x mm10.te.atac.idx
```

## Evaluation signals

- UMAP clustering computed on TE accessibility alone must recover the expected FACS-purified cell-type groupings.
- RLTR13A, RLTR4_Mm, RLTR13G and RMER19B/C must show elevated accessibility in CPCs, CD4+ T cells, ESCs and skin fibroblasts respectively.
- mm10.te.bed must contain only LINE, SINE, LTR and Retroposon classes with columns 6-8 and 11 retained.
- Each BAM processed with scTEATAC must produce per-cell counts referenced against the mm10.te.atac.idx index.
- Assembled matrix must export cleanly as .h5ad with cells from both mouse and human PBMC samples.

## Limitations

- TE information content is lower than that of genes; TE accessibility serves mainly as an addendum rather than a full replacement for gene/peak information.
- Using merged TE data as enhancer information in single cells remains speculative and was not demonstrated by the article.
- The TE classification is restricted to LINE, SINE, LTR and Retroposon repeat classes by the annotation filter.

## Evidence

- [methods] Filter repeat annotations and build ATAC index: "$ zcat mm10.te.txt.gz | grep -E 'LINE|SINE|LTR|Retroposon' | cut -f6-8,11 >mm10.te.bed"
- [methods] Build scATAC TE index command: "$ scTEATAC_build -g mm10.te.bed -o mm10.te.atac"
- [methods] Process scATAC BAM: "scTEATAC -i input.bam -x mm10.te.atac.idx"
- [abstract] Cell-type discrimination from TE accessibility: "Finally, we apply scTE to single-cell ATAC-seq data, and demonstrate that scTE can discriminate cell type using chromatin accessibly of TEs alone."
- [readme] Unfiltered BAM recommendation: "scTE makes BAM/SAM file as input, highly recommend to use unfiltered alignment file as input."
