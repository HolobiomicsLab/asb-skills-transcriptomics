---
name: genome-annotation-index-building
description: Use when preparing scTE (or scTEATAC) quantification for single-cell
  RNA/ATAC-seq alignments and a prebuilt index for the genome assembly (mm10, hg38,
  panTro6, macFas5, dm6, danRer11, xenTro9) is not yet available.
when_to_use_negative:
- Input is already a TE-by-cell count matrix or .h5ad/.csv from a previous scTE run
  — no index is needed at that stage.
- Reads are aligned to a genome assembly version different from the index assembly
  (e.g. mm9 or hg19 alignments against an mm10/hg38 index), causing coordinate mismatches.
- The goal is bulk RNA-seq gene/TE counting with te_counter, which builds its own
  indices via `te_genome` and does not consume scTE index files.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: scTE_build
  role: Builds the gene/TE genome index from GTF and TE BED/rmsk annotations, either
    by assembly name or via custom -te/-gene options
  repo: https://github.com/JiekaiLab/scTE
- name: scTEATAC_build
  role: Builds the TE-only index for single-cell ATAC-seq after filtering repeat classes
    to LINE/SINE/LTR/Retroposon
  repo: https://github.com/JiekaiLab/scTE
- name: scTE
  role: Consumes the built index via -x to allocate reads first to gene exons and
    then to TEs
  repo: https://github.com/JiekaiLab/scTE
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21808-x
    title: Identifying transposable element expression dynamics and heterogeneity
      during development at the single-cell level with a processing pipeline scTE
  source_content_hash: sha256:4b96386e8efce0900e5be028b1e5e5cea1398d8881491929e446bfe3b6d1ee4d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiekaiLab/scTE
  tools:
  - STARsolo
  - scTE
  - scTEATAC_build
  - scTE_build
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# genome-annotation-index-building

## Summary

Builds scTE genome indices that map reads to gene metagenes (from a GTF) and TE metagenes (from a RepeatMasker BED/rmsk file), enabling fast quantification of transposable element expression in single-cell data. It is the prerequisite step before running scTE on BAM/SAM alignments.

## When to use

Use when preparing scTE (or scTEATAC) quantification for single-cell RNA/ATAC-seq alignments and a prebuilt index for the genome assembly (mm10, hg38, panTro6, macFas5, dm6, danRer11, xenTro9) is not yet available.

## When NOT to use

- Input is already a TE-by-cell count matrix or .h5ad/.csv from a previous scTE run — no index is needed at that stage.
- Reads are aligned to a genome assembly version different from the index assembly (e.g. mm9 or hg19 alignments against an mm10/hg38 index), causing coordinate mismatches.
- The goal is bulk RNA-seq gene/TE counting with te_counter, which builds its own indices via `te_genome` and does not consume scTE index files.

## Inputs

- Gene annotation GTF (e.g. gencode.vM21.annotation.gtf.gz for mm10 or Ensembl release-103 GTF)
- RepeatMasker TE annotation: UCSC goldenPath rmsk.txt.gz, or a six-column BED (TEs.bed)
- Genome assembly version identifier (e.g. mm10, hg38, panTro6, macFas5, dm6, danRer11, xenTro9)

## Outputs

- scTE genome index file (e.g. mm10.exclusive.idx) passed to scTE via -x
- scTEATAC index (e.g. mm10.te.atac.idx) built from a class-filtered TE BED

## How to apply

For supported assemblies, run scTE_build with the assembly name (e.g. `scTE_build -g mm10`), which automatically downloads the paired annotations (GENCODE/Ensembl GTF plus UCSC goldenPath rmsk.txt.gz). For custom references, supply a six-column TE BED and a gene GTF with `scTE_build -te TEs.bed -gene Genes.gtf -o mm10`, producing an index such as mm10.exclusive.idx. The default exclusive mode encodes a hierarchy in which reads are allocated first to gene exons and then to TEs, so TEs inside exon/UTR regions only contribute to the gene; downstream alternatives (inclusive, nointron) are runtime scTE modes, not index changes. For scATAC-seq, first filter the rmsk table to LINE|SINE|LTR|Retroposon classes (grep -E plus cut -f6-8,11) and build with scTEATAC_build. The index version must match the genome assembly used for alignment, otherwise TE coordinates will be wrong.

## Related tools

- **scTE_build** (Builds the gene/TE genome index from GTF and TE BED/rmsk annotations, either by assembly name or via custom -te/-gene options) — https://github.com/JiekaiLab/scTE
- **scTEATAC_build** (Builds the TE-only index for single-cell ATAC-seq after filtering repeat classes to LINE/SINE/LTR/Retroposon) — https://github.com/JiekaiLab/scTE
- **scTE** (Consumes the built index via -x to allocate reads first to gene exons and then to TEs) — https://github.com/JiekaiLab/scTE

## Examples

```
scTE_build -te TEs.bed -gene Genes.gtf -o mm10
```

## Evaluation signals

- The expected index file exists and is named consistently with the assembly (e.g. mm10.exclusive.idx) and loads without error when passed to scTE -x.
- The TE BED is exactly six columns, as required for custom -te input.
- Downstream scTE output count matrix exists with cells as rows and gene/TE features as columns, confirming the index was usable.
- For scATAC-seq, the filtered TE BED contains only LINE/SINE/LTR/Retroposon classes after the grep/cut pipeline.
- Assembly version of the GTF and rmsk annotations matches the alignment genome (e.g. mm10 rmsk with mm10 BAM).

## Limitations

- Automated downloads in scTE_build pull fixed annotation releases (e.g. Gencode M21 for mouse, release-103 Ensembl GTFs), so newer assemblies or releases require the custom -te/-gene path.
- Only the listed assemblies (mm10, hg38, panTro6, macFas5, dm6, danRer11, xenTro9) have one-line index generation; other species need custom BED/GTF files.
- The index encodes the default exclusive read-allocation hierarchy; mode changes (inclusive/nointron) are applied at scTE runtime, not by rebuilding the index.

## Evidence

- [methods] Build genome annotation indices with scTE_build from TE BED and gene GTF files: [evidence span withheld — anchor status: not_found]
- [readme] custom index build command: "scTE_build -te TEs.bed -gene Genes.gtf -o custome"
- [readme] exclusive allocation hierarchy: "The scTE algorithm will allocate reads first to gene exons, and then to TEs by default."
- [readme] automatic annotation download for mouse: "http://hgdownload.soe.ucsc.edu/goldenPath/mm10/database/rmsk.txt.gz"
- [methods] ATAC-seq repeat class filter: "zcat mm10.te.txt.gz | grep -E 'LINE|SINE|LTR|Retroposon' | cut -f6-8,11 >mm10.te.bed"
