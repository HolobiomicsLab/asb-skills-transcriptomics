---
name: differential-te-expression-testing-between-cell-types
description: Use when single-cell BAM/SAM files from distinct cell types (e.g. ESCs
  and MEFs) need per-cell TE counts from scTE and a list of cell-type-specific differentially
  expressed TEs.
when_to_use_negative:
- Input is already a per-cell TE count matrix or a DE results table rather than raw
  aligned BAM/SAM reads
- The goal is TE detection from scATAC-seq chromatin accessibility data — use scTEATAC/scTEATAC_build
  instead of scTE counting
- Reads overlapping intronic TE loci are expected to be biologically meaningful TE
  expression rather than background gene signal — the exclusive or inclusive modes
  may be more appropriate than nointron
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0797
tools:
- name: scTE_build
  role: Builds the mm10.exclusive.idx genome index from the mm10 rmsk TE annotation
    and Ensembl/GENCODE gene GTF before TE quantification
  repo: https://github.com/JiekaiLab/scTE
- name: scTE
  role: Quantifies per-cell TE counts from BAM/SAM input in exclusive or nointron
    mode, using CR/UR or CB/UB barcode and UMI tags
  repo: https://github.com/JiekaiLab/scTE
- name: Cell Ranger
  role: Source of 10x-style BAM files whose reads carry CB/UB cell barcode and UMI
    tags passed to scTE
- name: STARsolo
  role: Source of 10x-style BAM files whose reads carry CR/UR cell barcode and UMI
    tags passed to scTE
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21808-x
    title: Identifying transposable element expression dynamics and heterogeneity
      during development at the single-cell level with a processing pipeline scTE
  source_content_hash: sha256:f1375816ca1f1ec0d3b49629819c9c1b3119662ba2440028b86725a26cd7ba4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiekaiLab/scTE
  tools:
  - Cell Ranger
  - STARsolo
  - scTE
  - scTE_build
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# differential-te-expression-testing-between-cell-types

## Summary

Quantify transposable element (TE) expression per cell with scTE, aggregate the per-cell TE count matrix by cell type (e.g. ESC vs MEF), and test for differentially expressed TEs using a ≥2-fold change with FDR < 0.05, optionally re-testing under the nointron mode to confirm TE specificity is not driven by intronic gene reads.

## When to use

Use when single-cell BAM/SAM files from distinct cell types (e.g. ESCs and MEFs) need per-cell TE counts from scTE and a list of cell-type-specific differentially expressed TEs.

## When NOT to use

- Input is already a per-cell TE count matrix or a DE results table rather than raw aligned BAM/SAM reads
- The goal is TE detection from scATAC-seq chromatin accessibility data — use scTEATAC/scTEATAC_build instead of scTE counting
- Reads overlapping intronic TE loci are expected to be biologically meaningful TE expression rather than background gene signal — the exclusive or inclusive modes may be more appropriate than nointron

## Inputs

- Unfiltered 10x-style scRNA-seq BAM/SAM alignment files (e.g. GSE103221) with cell barcode and UMI tags (CR/UR or CB/UB)
- mm10 UCSC rmsk RepeatMasker TE annotation (rmsk.txt.gz)
- Ensembl/GENCODE gene annotation GTF for mm10

## Outputs

- Per-cell TE count matrix (optionally HDF5) from scTE
- Differential TE expression results table with fold change and FDR per TE (de_te_results.csv)
- List of cell-type-specific DE TEs passing ≥2-fold change and FDR < 0.05, with ESC-enriched examples such as ERVB7_1-LTR_MM and MEF-enriched examples such as RMER10B

## How to apply

Build a mouse exclusive-mode index with scTE_build from the mm10 UCSC rmsk RepeatMasker TE annotation and the Ensembl/GENCODE gene GTF, producing mm10.exclusive.idx; scTE allocates reads first to gene exons and then to TEs by default. Run scTE on the unfiltered 10x-style BAM files (GSE103221), passing the correct cell-barcode/UMI tags (-CB CR -UMI UR for STARsolo, -CB CB -UMI UB for Cell Ranger). Aggregate the per-cell TE count matrix by cell type (ESC vs MEF) and perform differential expression testing on TE counts, selecting TEs with ≥2-fold change and FDR < 0.05. As a robustness check, re-run scTE with '--mode/-m nointron' (which removes TEs inside gene introns from the TE score) and re-run the differential test; in the reference analysis 94 of 108 TEs stayed specific and only 14 were altered, with ERVB7_1-LTR_MM enriched in ESCs and RMER10B enriched in MEFs. Write all DE results to de_te_results.csv.

## Related tools

- **scTE_build** (Builds the mm10.exclusive.idx genome index from the mm10 rmsk TE annotation and Ensembl/GENCODE gene GTF before TE quantification) — https://github.com/JiekaiLab/scTE
- **scTE** (Quantifies per-cell TE counts from BAM/SAM input in exclusive or nointron mode, using CR/UR or CB/UB barcode and UMI tags) — https://github.com/JiekaiLab/scTE
- **Cell Ranger** (Source of 10x-style BAM files whose reads carry CB/UB cell barcode and UMI tags passed to scTE)
- **STARsolo** (Source of 10x-style BAM files whose reads carry CR/UR cell barcode and UMI tags passed to scTE) — https://github.com/alexdobin/STAR

## Examples

```
scTE -i inp.bam -o out -x mm10.exclusive.idx --hdf5 True -CB CB -UMI UB -m nointron
```

## Evaluation signals

- Exclusive-mode differential testing on GSE103221 ESC vs MEF data reproduces 108 significantly DE TEs at ≥2-fold change and FDR < 0.05
- Re-testing in nointron mode confirms the majority remain specific: 94 of 108 TEs stay specific, only 14 are altered
- Known biology is recovered: ERVB7_1-LTR_MM enriched in ESCs and RMER10B enriched in MEFs among the DE TEs
- Per-cell TE counts are populated only for cell barcodes present in the unfiltered BAM, with barcode/UMI tags correctly matched to the aligner (CR/UR for STARsolo, CB/UB for Cell Ranger)
- All DE results are written to de_te_results.csv with fold-change and FDR columns for every tested TE

## Limitations

- scTE by default allocates reads first to gene exons and then to TEs, so TEs inside exon/UTR regions of annotated genes contribute to the gene and not the TE score, which can mask exonic TE signal
- Differential TE calls can be sensitive to reads from TEs inside gene introns; only 94 of 108 cell-type-specific TEs remained specific when intronic reads were removed in nointron mode
- scTE should be run on unfiltered alignment files; filtered BAMs may omit valid cell barcodes
- TE information content is lower than that of genes; TEs serve mainly as an addendum to gene information

## Evidence

- [other] Exclusive-mode threshold and nointron robustness check: [evidence span withheld — anchor status: not_found]
- [other] Nointron-mode confirmation counts: [evidence span withheld — anchor status: not_found]
- [methods] Read allocation order in scTE: "The scTE algorithm will allocate reads first to gene exons, and then to TEs by default."
- [readme] Unfiltered BAM input requirement: "scTE makes BAM/SAM file as input, highly recommend to use unfiltered alignment file as input."
- [readme] Nointron mode definition: "If you want to remove the TEs inside the intron 
of genes, you can sete `–mode/-m nointron` in scTE"
