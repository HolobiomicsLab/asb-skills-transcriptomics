---
name: scte-mode-comparison-validation
description: Use when scTE differential TE calls (e.g. ≥2-fold change, FDR < 0.05
  between ESCs and MEFs) need robustness checks against TE annotation mode, especially
  to test whether intronic TE reads drive specificity.
when_to_use_negative:
- Input is already an aggregated TE count or DE results table — the skill requires
  re-running scTE from raw BAM/SAM alignments in two modes
- The analysis aims to assign reads overlapping TE-in-exon/UTR regions to both genes
  and TEs — that requires the inclusive mode (-m inclusive), not the exclusive/nointron
  comparison
- Data are single-cell ATAC-seq; TE accessibility should use scTEATAC/scTEATAC_build
  rather than scTE count modes
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0621
tools:
- name: scTE_build
  role: Builds the mm10.exclusive.idx genome index from TE BED (rmsk-derived) and
    gene GTF annotations used for both modes
  repo: https://github.com/JiekaiLab/scTE
- name: scTE
  role: Quantifies TE expression from BAM/SAM per cell; run in default exclusive mode
    and re-run with -m nointron to compare DE TE calls
  repo: https://github.com/JiekaiLab/scTE
- name: STARsolo
  role: Upstream aligner whose BAMs carry CR/UR cell barcode and UMI tags consumed
    by scTE via -CB CR -UMI UR
  repo: https://github.com/alexdobin/STAR
- name: Cell Ranger
  role: Alternative 10x aligner whose BAMs carry CB/UB tags consumed by scTE via -CB
    CB -UMI UB
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21808-x
    title: Identifying transposable element expression dynamics and heterogeneity
      during development at the single-cell level with a processing pipeline scTE
  source_content_hash: sha256:d1b4ba540fe66fd93a0220bff3478561a2a9b343902f6ad4a8d4b15ceca68085
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiekaiLab/scTE
  tools:
  - Cell Ranger
  - STARsolo
  - scTE
  - scTE_build
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# scte-mode-comparison-validation

## Summary

A validation skill for scTE-based TE differential expression: re-run scTE in a second annotation mode (nointron vs default exclusive) and quantify how many significant DE TEs retain cell-type specificity, testing whether findings depend on intronic reads.

## When to use

Use when scTE differential TE calls (e.g. ≥2-fold change, FDR < 0.05 between ESCs and MEFs) need robustness checks against TE annotation mode, especially to test whether intronic TE reads drive specificity.

## When NOT to use

- Input is already an aggregated TE count or DE results table — the skill requires re-running scTE from raw BAM/SAM alignments in two modes
- The analysis aims to assign reads overlapping TE-in-exon/UTR regions to both genes and TEs — that requires the inclusive mode (-m inclusive), not the exclusive/nointron comparison
- Data are single-cell ATAC-seq; TE accessibility should use scTEATAC/scTEATAC_build rather than scTE count modes

## Inputs

- Unfiltered 10x-style BAM/SAM alignment files (e.g. GSE103221 ESC/MEF data) with cell barcode and UMI tags (CR/UR for STARsolo, CB/UB for Cell Ranger)
- mm10 UCSC rmsk RepeatMasker TE annotation (rmsk.txt.gz)
- Ensembl/GENCODE mouse gene annotation GTF
- Cell-type labels (ESC vs MEF) for aggregating per-cell TE count matrices

## Outputs

- Per-cell TE count matrices in HDF5 format for exclusive and nointron modes
- Differential TE expression table (de_te_results.csv) with fold change and FDR per TE per mode
- Mode-comparison summary of retained vs altered cell-type-specific TEs (94 retained, 14 altered of 108)

## How to apply

Build the mouse exclusive-mode index (mm10.exclusive.idx) with scTE_build from the mm10 UCSC rmsk RepeatMasker TE annotation and Ensembl/GENCODE gene GTF, then run scTE on unfiltered 10x-style BAM/SAM files, allocating reads first to gene exons and then to TEs. Use -CB CR -UMI UR for STARsolo outputs or -CB CB -UMI UB for Cell Ranger outputs, with --hdf5 True to obtain count matrices. Aggregate per-cell TE counts by cell type (ESC vs MEF) and call differentially expressed TEs with ≥2-fold change and FDR < 0.05, verifying the exclusive-mode baseline of 108 DE TEs (e.g. ERVB7_1-LTR_MM in ESCs, RMER10B in MEFs). Re-run with --mode nointron, which removes TEs inside gene introns from the TE score, and repeat the differential test; judge success by how many of the 108 TEs remain specific (expected: 94 of 108, with only 14 altered), writing results to de_te_results.csv.

## Related tools

- **scTE_build** (Builds the mm10.exclusive.idx genome index from TE BED (rmsk-derived) and gene GTF annotations used for both modes) — https://github.com/JiekaiLab/scTE
- **scTE** (Quantifies TE expression from BAM/SAM per cell; run in default exclusive mode and re-run with -m nointron to compare DE TE calls) — https://github.com/JiekaiLab/scTE
- **STARsolo** (Upstream aligner whose BAMs carry CR/UR cell barcode and UMI tags consumed by scTE via -CB CR -UMI UR) — https://github.com/alexdobin/STAR
- **Cell Ranger** (Alternative 10x aligner whose BAMs carry CB/UB tags consumed by scTE via -CB CB -UMI UB)

## Examples

```
scTE -i inp.bam -o out -x mm10.exclusive.idx --hdf5 True -CB CR -UMI UR; scTE -i inp.bam -o out_nointron -x mm10.exclusive.idx -m nointron --hdf5 True -CB CR -UMI UR
```

## Evaluation signals

- Exclusive-mode run reproduces 108 significantly DE TEs (≥2-fold change, FDR < 0.05) between ESCs and MEFs
- Known markers are enriched in the expected direction: ERVB7_1-LTR_MM in ESCs and RMER10B in MEFs
- Nointron-mode rerun retains 94 of the 108 TEs as cell-type-specific, with only 14 altered
- Both runs used the unfiltered BAM as scTE input (not the Cell Ranger filtered file) and correct barcode/UMI tags for the aligner
- Output de_te_results.csv contains per-TE fold change and FDR for both modes with matching TE identifiers

## Limitations

- By default (exclusive mode) TEs inside exon/UTR regions of GENCODE-annotated genes only contribute to the gene, not the TE score, so results depend on gene annotation completeness
- Nointron mode removes TEs inside gene introns from the TE score, which can change specificity for intron-residing TEs (14 of 108 altered here)
- scTE recommends unfiltered alignment files as input; using filtered BAMs may change cell and UMI recovery
- TE information content is lower than that of genes, so TE counts are sparse and serve mainly as an addendum to gene information

## Evidence

- [methods] The scTE algorithm will allocate reads first to gene exons, and then to TEs by default.: "The scTE algorithm will allocate reads first to gene exons, and then to TEs by default."
- [methods] nointron mode removes intronic TEs: "If you want to remove the TEs inside the intron of genes, you can sete `–mode/-m nointron` in scTE"
- [methods] STARsolo BAM command line: "$ scTE -i inp.bam -o out -x mm10.exclusive.idx --hdf5 True -CB CR -UMI UR"
- [methods] unfiltered BAM recommendation: "scTE makes BAM/SAM file as input, highly recommend to use unfiltered alignment file as input."
- [results] 94 of 108 TEs remain specific in nointron mode: [evidence span withheld — anchor status: not_found]
