---
name: te-index-construction
description: 'Use when starting a scTE-based analysis: you have a genome assembly
  (mm10, hg38, panTro6, macFas5, dm6, danRer11, xenTro9) and need a TE index from
  RepeatMasker rmsk annotation, or a custom TE BED plus gene GTF, before counting
  reads.'
when_to_use_negative:
- The index for the target assembly already exists and is up to date — rebuilding
  is unnecessary and the existing .idx can be passed directly to scTE/scTEATAC via
  -x
- The task is standard gene-level quantification without any TE component — a conventional
  aligner/counting pipeline suffices
- The input is already a per-cell TE count or accessibility matrix — index construction
  operates on genome annotations, not expression data
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: scTE
  role: Pipeline whose genome indices (scTE_build -g assembly or scTE_build -te TEs.bed
    -gene Genes.gtf -o custom) are built here for scRNA-seq TE quantification
  repo: https://github.com/JiekaiLab/scTE
- name: scTE_build
  role: Builds gene+TE indices from automatically downloaded GENCODE GTF and UCSC
    rmsk annotations, or from custom BED/GTF files
  repo: https://github.com/JiekaiLab/scTE
- name: scTEATAC_build
  role: Builds the TE-only index for scATAC-seq from the filtered LINE/SINE/LTR/Retroposon
    TE BED (scTEATAC_build -g mm10.te.bed -o mm10.te.atac)
  repo: https://github.com/JiekaiLab/scTE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21808-x
    title: Identifying transposable element expression dynamics and heterogeneity
      during development at the single-cell level with a processing pipeline scTE
  source_content_hash: sha256:979f10ba68d05e73031566e09e067fb371b483c835c36ac3d38d8314d8ad786d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiekaiLab/scTE
  tools:
  - scTE
  - scTEATAC
  - scTEATAC_build
  - scTE_build
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# te-index-construction

## Summary

Constructs scTE/scTEATAC alignment indices from UCSC RepeatMasker (rmsk) repeat annotations and gene GTF files so that single-cell RNA-seq or ATAC-seq reads can be quantified over transposable elements. This is the prerequisite indexing step that makes per-cell TE count matrices possible.

## When to use

Use when starting a scTE-based analysis: you have a genome assembly (mm10, hg38, panTro6, macFas5, dm6, danRer11, xenTro9) and need a TE index from RepeatMasker rmsk annotation, or a custom TE BED plus gene GTF, before counting reads.

## When NOT to use

- The index for the target assembly already exists and is up to date — rebuilding is unnecessary and the existing .idx can be passed directly to scTE/scTEATAC via -x
- The task is standard gene-level quantification without any TE component — a conventional aligner/counting pipeline suffices
- The input is already a per-cell TE count or accessibility matrix — index construction operates on genome annotations, not expression data

## Inputs

- UCSC goldenPath RepeatMasker rmsk.txt.gz annotation for the genome assembly (e.g. http://hgdownload.soe.ucsc.edu/goldenPath/mm10/database/rmsk.txt.gz)
- Gene annotation GTF (e.g. GENCODE vM21 for mouse, GENCODE v30 for human, Ensembl release-103 GTFs for other species)
- Custom six-column TE BED file (optional, for custom references)
- Genome assembly identifier (mm10, hg38, panTro6, macFas5, dm6, danRer11, xenTro9)

## Outputs

- scTE genome index (e.g. mm10.exclusive.idx) mapping reads to genes and TEs
- scATAC-seq TE index (e.g. mm10.te.atac.idx) built from the filtered TE BED
- Filtered TE BED file (mm10.te.bed) restricted to LINE/SINE/LTR/Retroposon classes

## How to apply

For supported assemblies, run scTE_build with the assembly name (e.g. scTE_build -g mm10); the tool automatically downloads the GENCODE GTF (e.g. gencode.vM21.annotation.gtf.gz for mouse) and the UCSC goldenPath rmsk.txt.gz file and builds the index. For scATAC-seq, first filter the rmsk annotation to LINE, SINE, LTR and Retroposon classes keeping columns 6-8 and 11 into a TE BED (zcat mm10.te.txt.gz | grep -E 'LINE|SINE|LTR|Retroposon' | cut -f6-8,11 > mm10.te.bed), then build the ATAC index with scTEATAC_build -g mm10.te.bed -o mm10.te.atac. For custom references, supply a six-column BED for TEs (-te) and a GTF for genes (-gene) with the -o output prefix. Rationale: scTE allocates reads to gene exons first and then to TEs, so the index encodes exon/UTR overlap rules (controllable via inclusive/nointron modes) that determine whether TEs inside genes receive counts.

## Related tools

- **scTE** (Pipeline whose genome indices (scTE_build -g assembly or scTE_build -te TEs.bed -gene Genes.gtf -o custom) are built here for scRNA-seq TE quantification) — https://github.com/JiekaiLab/scTE
- **scTE_build** (Builds gene+TE indices from automatically downloaded GENCODE GTF and UCSC rmsk annotations, or from custom BED/GTF files) — https://github.com/JiekaiLab/scTE
- **scTEATAC_build** (Builds the TE-only index for scATAC-seq from the filtered LINE/SINE/LTR/Retroposon TE BED (scTEATAC_build -g mm10.te.bed -o mm10.te.atac)) — https://github.com/JiekaiLab/scTE

## Examples

```
zcat mm10.te.txt.gz | grep -E 'LINE|SINE|LTR|Retroposon' | cut -f6-8,11 > mm10.te.bed; scTEATAC_build -g mm10.te.bed -o mm10.te.atac
```

## Evaluation signals

- The downloaded rmsk.txt.gz matches the intended assembly (mm10, hg38, etc.) and unzips to a valid tabular annotation
- The filtered mm10.te.bed contains only LINE, SINE, LTR and Retroposon repeat classes and has the expected four retained fields (columns 6-8 and 11)
- scTE_build / scTEATAC_build complete without error and produce the expected index file (e.g. mm10.exclusive.idx, mm10.te.atac.idx) consumable via -x by scTE/scTEATAC
- Custom TE BED is six columns in UCSC BED format, as required by scTE_build -te

## Limitations

- Automated annotation download only covers the supported assemblies (mm10, hg38, panTro6, macFas5, dm6, danRer11, xenTro9); other genomes require custom -te/-gene inputs
- By default TEs inside exon/UTR regions of GENCODE-annotated genes contribute to the gene score and not the TE score; changing this requires the inclusive or nointron modes rather than re-indexing alone
- The scATAC-seq index path requires manual filtering of rmsk to LINE/SINE/LTR/Retroposon classes; unfiltered rmsk is not used directly for scTEATAC_build

## Evidence

- [readme] scTE_build generates genome indices automatically: "scTE builds genome indices for the fast alignment of reads to genes and TEs. These indices can be automatically generated using the commands:"
- [methods] Filter rmsk to LINE/SINE/LTR/Retroposon for ATAC: "$ zcat mm10.te.txt.gz | grep -E 'LINE|SINE|LTR|Retroposon' | cut -f6-8,11 >mm10.te.bed"
- [methods] scTEATAC_build command: "$ scTEATAC_build -g mm10.te.bed -o mm10.te.atac"
- [readme] Read allocation rule encoded by index: "The scTE algorithm will allocate reads first to gene exons, and then to TEs by default. Hence TEs inside exon/UTR regions of genes annotated"
- [readme] Custom reference input formats: "-te
    Six columns bed file for transposable elements annotation.
-gene
    Gtf file for genes annotation."
