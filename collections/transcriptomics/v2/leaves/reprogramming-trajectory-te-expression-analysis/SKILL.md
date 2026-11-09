---
name: reprogramming-trajectory-te-expression-analysis
description: Use when analyzing single-cell RNA-seq of OKSM reprogramming (e.g. GSE115943)
  to test whether specific TE families are expressed in a branch-restricted manner
  across intermediate cell fates, or to identify TEs upregulated specifically in the
  successful reprogramming route.
when_to_use_negative:
- Input is already a gene-by-cell expression matrix or TE count table — scTE requires
  aligned BAM/SAM reads with barcode/UMI tags as input
- Only a filtered cell-by-gene matrix is available and the unfiltered alignment BAM
  cannot be regenerated, since scTE highly recommends unfiltered alignment files as
  input
- The question concerns bulk RNA-seq TE quantification rather than single-cell branch-resolved
  dynamics
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: scTE_build
  role: Builds the mm10.exclusive.idx index from TE BED and gene GTF annotations;
    default exclusive mode allocates reads to gene exons first, then TEs
  repo: https://github.com/JiekaiLab/scTE
- name: scTE
  role: Quantifies per-cell TE expression from the reprogramming BAM using the exclusive
    index, writing the result as an .h5ad matrix with --hdf5 True
  repo: https://github.com/JiekaiLab/scTE
- name: Cell Ranger
  role: Produces the source BAM files for GSE115943, with cell barcodes and UMIs stored
    in CB:Z/UB:Z tags
- name: STARsolo
  role: Alternative aligner producing BAMs with cell barcodes and UMIs integrated
    into CR:Z/UR:Z tags
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21808-x
    title: Identifying transposable element expression dynamics and heterogeneity
      during development at the single-cell level with a processing pipeline scTE
  source_content_hash: sha256:1fddbe18f71c42fb68ebd1e400c8755c303f61a91b456caf0552c0922de90400
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiekaiLab/scTE
  tools:
  - Cell Ranger
  - STARsolo
  - scTE
  - scTE_build
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# reprogramming-trajectory-te-expression-analysis

## Summary

Quantifies branch-restricted transposable element expression during OKSM somatic cell reprogramming by running scTE in exclusive mode on unfiltered Cell Ranger/STARsolo BAM files (mm10), then tracing TE family trajectories (e.g. ERVB7_1-LTR_MM, IAPEz-int, RLTR4_Mm, Lx) across iPSC, trophoblast, stromal, and neural-like intermediate branches.

## When to use

Use when analyzing single-cell RNA-seq of OKSM reprogramming (e.g. GSE115943) to test whether specific TE families are expressed in a branch-restricted manner across intermediate cell fates, or to identify TEs upregulated specifically in the successful reprogramming route.

## When NOT to use

- Input is already a gene-by-cell expression matrix or TE count table — scTE requires aligned BAM/SAM reads with barcode/UMI tags as input
- Only a filtered cell-by-gene matrix is available and the unfiltered alignment BAM cannot be regenerated, since scTE highly recommends unfiltered alignment files as input
- The question concerns bulk RNA-seq TE quantification rather than single-cell branch-resolved dynamics

## Inputs

- Unfiltered Cell Ranger or STARsolo BAM files from OKSM reprogramming scRNA-seq (e.g. GSE115943) with cell barcode and UMI in CB:Z/UB:Z or CR:Z/UR:Z tags
- mm10 gene annotation GTF from Ensembl
- RepeatMasker rmsk.txt.gz table from UCSC goldenPath (http://hgdownload.soe.ucsc.edu/goldenPath/mm10/database/rmsk.txt.gz)
- Six-column TE BED file converted from the rmsk table

## Outputs

- scTE_gse115943_counts.h5ad with TE and cell annotations
- te_branch_summary.csv summarizing branch-specific TE expression across iPSC, trophoblast, stromal, and neural-like branches
- mm10.exclusive.idx scTE reference index

## How to apply

Build an mm10 scTE reference index with scTE_build -te TEs.bed -gene Genes.gtf -o mm10, keeping the default exclusive mode so reads are allocated first to gene exons and only then to TEs (do NOT set -m inclusive or -m nointron), using the Ensembl mm10 GTF and the UCSC goldenPath rmsk.txt.gz RepeatMasker table. Obtain the unfiltered GSE115943 BAM files and verify cell barcodes and UMIs are stored in CB:Z/UB:Z (Cell Ranger) or CR:Z/UR:Z (STARsolo) tags. Run scTE -i reprogramming.bam -o scte_gse115943 -x mm10.exclusive.idx --hdf5 True -CB CB -UMI UB to produce a per-cell TE count matrix in .h5ad format. Extract trajectories of the target TE families — ERVB7_1-LTR_MM and RLTR13G for the successful iPSC route initiating at the MET and peaking at the iPSC stage; IAPEz-int and IAPLTR1_Mm for the trophoblast branch; RLTR4_Mm for the stromal branch; Lx for the neural-like branch — and summarize expression across reprogramming branches in te_branch_summary.csv.

## Related tools

- **scTE_build** (Builds the mm10.exclusive.idx index from TE BED and gene GTF annotations; default exclusive mode allocates reads to gene exons first, then TEs) — https://github.com/JiekaiLab/scTE
- **scTE** (Quantifies per-cell TE expression from the reprogramming BAM using the exclusive index, writing the result as an .h5ad matrix with --hdf5 True) — https://github.com/JiekaiLab/scTE
- **Cell Ranger** (Produces the source BAM files for GSE115943, with cell barcodes and UMIs stored in CB:Z/UB:Z tags)
- **STARsolo** (Alternative aligner producing BAMs with cell barcodes and UMIs integrated into CR:Z/UR:Z tags) — https://github.com/alexdobin/STAR

## Examples

```
scTE -i reprogramming.bam -o scte_gse115943 -x mm10.exclusive.idx --hdf5 True -CB CB -UMI UB
```

## Evaluation signals

- scTE_gse115943_counts.h5ad exists and contains both TE and cell annotations
- te_branch_summary.csv reproduces the expected branch-specific enrichment/silencing pattern: ERVB7_1-LTR_MM in iPSCs, IAPEz-int/IAPLTR1_Mm in the trophoblast branch, RLTR4_Mm in the stromal branch, and Lx in the neural-like branch
- ERVB7_1-LTR_MM, ETnERV-int, and RLTR13G show upregulation specifically in the successful reprogramming route, initiating at the MET and peaking at the iPSC stage
- The index was built in exclusive (default) mode, i.e. no -m inclusive or -m nointron flag was used, so TEs inside exon/UTR regions of GENCODE genes contribute to the gene score only
- Barcode/UMI tag names passed via -CB/-UMI match the actual BAM tags (CB/UB for Cell Ranger, CR/UR for STARsolo)

## Limitations

- scTE highly recommends using the unfiltered alignment file as input; filtered BAMs may lose valid barcoded reads
- TE information content is lower than that of genes; TE data serve mainly as an addendum to gene information
- Read allocation mode changes interpretation: exclusive mode discards TEs overlapping gene exons/UTRs from TE counts, and -m nointron removes intronic TEs
- How TEs are involved in stem cell differentiation and somatic cell reprogramming is still largely unknown, so TE trajectories are descriptive rather than causal

## Evidence

- [methods] scTE makes BAM/SAM file as input, highly recommend to use unfiltered alignment file as input.: "scTE makes BAM/SAM file as input, highly recommend to use unfiltered alignment file as input."
- [methods] Exclusive read allocation: "The scTE algorithm will allocate 
reads first to gene exons, and then to TEs by default."
- [methods] Example scTE invocation with hdf5 output: "$ scTE -i inp.bam -o out -x mm10.exclusive.idx --hdf5 True -CB CR -UMI UR"
- [methods] Cell barcode tag requirements for STARsolo BAMs: "For `bam` file generated by [STARsolo](https://github.com/alexdobin/STAR) etc, the cell barcodes and UMI need to be integrated into the read 'CR:Z' or 'UR:Z' tage"
- [readme] README command line for building a custom scTE index: "scTE_build -te TEs.bed -gene Genes.gtf -o custome"
- [abstract] Branch-restricted TE expression finding during reprogramming: "Specific TE types are expressed in subpopulations of embryonic stem cells and are dynamically regulated during pluripotency reprogramming, differentiation, and embryogenesis."
