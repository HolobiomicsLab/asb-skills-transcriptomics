---
name: repeatmasker-annotation-filtering
description: Use when building a custom TE reference for scTE or scTEATAC and a genome
  lacks a prebuilt scTE index — e.g. mm10 rmsk.txt must be subset to LINE/SINE/LTR/Retroposon
  classes before index construction.
when_to_use_negative:
- The target genome already has an official prebuilt scTE index (scTE_build -g mm10
  / -g hg38) that can be generated automatically — filtering is only needed for custom
  references
- The analysis requires all RepeatMasker classes (e.g. Simple_repeats, Low_complexity,
  Satellite) rather than only LINE, SINE, LTR and Retroposon classes
- Input is already a six-column TE BED or an existing scTE/scTEATAC index file — no
  re-filtering of rmsk is required
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: scTE_build
  role: Consumes the filtered six-column TE BED together with a gene GTF (-te/-gene
    options) to build the scTE genome index for single-cell RNA-seq TE quantification
  repo: https://github.com/JiekaiLab/scTE
- name: scTEATAC_build
  role: Consumes the filtered TE BED (-g) to build the scATAC-seq TE accessibility
    index used by scTEATAC
  repo: https://github.com/JiekaiLab/scTE
- name: scTE
  role: Downstream tool that quantifies per-cell TE expression/accessibility from
    BAM using the index built from the filtered annotation
  repo: https://github.com/JiekaiLab/scTE
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21808-x
    title: Identifying transposable element expression dynamics and heterogeneity
      during development at the single-cell level with a processing pipeline scTE
  source_content_hash: sha256:6334e133ded3d701ddf84ac87efc2f51e80c2107c096645f2f513c8eea076353
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiekaiLab/scTE
  tools:
  - STARsolo
  - scTE
  - scTEATAC
  - scTEATAC_build
  - scTE_build
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# repeatmasker-annotation-filtering

## Summary

Convert UCSC RepeatMasker (rmsk) repeat annotations into a filtered, six-column BED of transposable elements restricted to LINE, SINE, LTR and Retroposon classes, suitable as TE input for scTE_build and scTEATAC_build genome indices.

## When to use

Use when building a custom TE reference for scTE or scTEATAC and a genome lacks a prebuilt scTE index — e.g. mm10 rmsk.txt must be subset to LINE/SINE/LTR/Retroposon classes before index construction.

## When NOT to use

- The target genome already has an official prebuilt scTE index (scTE_build -g mm10 / -g hg38) that can be generated automatically — filtering is only needed for custom references
- The analysis requires all RepeatMasker classes (e.g. Simple_repeats, Low_complexity, Satellite) rather than only LINE, SINE, LTR and Retroposon classes
- Input is already a six-column TE BED or an existing scTE/scTEATAC index file — no re-filtering of rmsk is required

## Inputs

- UCSC RepeatMasker rmsk.txt.gz annotation for the target genome assembly (e.g. mm10, hg38, panTro6, macFas5, dm6, danRer11)
- Gene annotation GTF (e.g. GENCODE or Ensembl release) when building a combined scTE_build index

## Outputs

- Six-column BED file of filtered TE coordinates (mm10.te.bed) with columns 6-8 and 11 of the rmsk table (chromosome, start, end, repeat name)
- scTE genome index (e.g. mm10.exclusive.idx) or scATAC-seq TE index (e.g. mm10.te.atac.idx) built from the filtered BED

## How to apply

Download the RepeatMasker annotation for the target assembly from UCSC goldenPath (e.g. wget -c http://hgdownload.soe.ucsc.edu/goldenPath/mm10/database/rmsk.txt.gz -O mm10.te.txt.gz). Filter the raw rmsk table with grep -E 'LINE|SINE|LTR|Retroposon' to retain only these repeat classes, then extract columns 6-8 (chromosome, start, end) plus column 11 (repeat name) with cut, producing a six-column BED (mm10.te.bed) as scTE requires. This filtered BED is then consumed by scTEATAC_build (-g mm10.te.bed -o mm10.te.atac) for scATAC-seq TE accessibility indices, or by scTE_build (-te TEs.bed -gene Genes.gtf) for scRNA-seq exclusive/inclusive quantification indices. The rationale is that scTE's default exclusive mode allocates reads first to gene exons and then to TEs, so the TE BED must be format-compatible with the GENCODE GTF used for the gene index.

## Related tools

- **scTE_build** (Consumes the filtered six-column TE BED together with a gene GTF (-te/-gene options) to build the scTE genome index for single-cell RNA-seq TE quantification) — https://github.com/JiekaiLab/scTE
- **scTEATAC_build** (Consumes the filtered TE BED (-g) to build the scATAC-seq TE accessibility index used by scTEATAC) — https://github.com/JiekaiLab/scTE
- **scTE** (Downstream tool that quantifies per-cell TE expression/accessibility from BAM using the index built from the filtered annotation) — https://github.com/JiekaiLab/scTE

## Examples

```
zcat mm10.te.txt.gz | grep -E 'LINE|SINE|LTR|Retroposon' | cut -f6-8,11 > mm10.te.bed; scTEATAC_build -g mm10.te.bed -o mm10.te.atac
```

## Evaluation signals

- Output mm10.te.bed has exactly six tab-separated columns per line (chr, start, end, plus rmsk fields incl. repeat name), matching scTE_build's '-te: Six columns bed file for transposable elements annotation' requirement
- grep -c on the BED returns counts only for LINE, SINE, LTR and Retroposon classes; no Simple_repeat/Low_complexity/Satellite rows remain
- scTEATAC_build or scTE_build completes without a BED-format error and produces a .idx index file
- Downstream UMAP/clustering on TE accessibility alone recovers expected FACS-purified cell types, confirming the annotation fed the index correctly

## Limitations

- The class filter is hardcoded to LINE|SINE|LTR|Retroposon; other repeat classes are excluded and TE fragments overlapping gene exons/UTRs are assigned to the gene by default in exclusive mode
- The technique is demonstrated for prebuilt-supported assemblies (mm10, hg38, panTro6, macFas5, dm6, danRer11, xenTro9); custom references require careful format checking of the BED

## Evidence

- [methods] Filter repeat annotations to LINE, SINE, LTR and Retroposon classes and build scATAC-seq index: "$ zcat mm10.te.txt.gz | grep -E 'LINE|SINE|LTR|Retroposon' | cut -f6-8,11 >mm10.te.bed"
- [methods] scTEATAC_build consumes the filtered BED to build the ATAC index: "$ scTEATAC_build -g mm10.te.bed -o mm10.te.atac"
- [methods] Download source of the rmsk annotation from UCSC goldenPath: "$ wget -c http://hgdownload.soe.ucsc.edu/goldenPath/mm10/database/rmsk.txt.gz -O mm10.te.txt.gz"
- [readme] scTE_build requires a six-column TE BED as input for custom references: "-te
    Six columns bed file for transposable elements annotation."
- [readme] Custom scTE_build index construction from TE BED and gene GTF: "scTE_build -te TEs.bed -gene Genes.gtf -o custome"
