---
name: differential-te-expression-across-cardiac-lineage-branches
description: Use when single-cell RNA-seq BAMs from a branching differentiation system
  need TE-level quantification and comparison of TE expression between annotated cell-fate
  branches (e.g., contracting vs non-contractile cardiomyocyte lineages).
when_to_use_negative:
- Input is already a gene or TE feature count table (e.g., a matrix from te_counter)
  rather than aligned BAM/SAM reads — scTE requires alignment files as input.
- Only bulk RNA-seq data are available; TE heterogeneity between single-cell branches
  cannot be assessed without single-cell barcoded libraries.
- The organism/genome is not among the assemblies scTE_build supports (mm10, hg38,
  panTro6, macFas5, dm6, danRer11, xenTro9) and no custom BED/GTF is available.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3170
tools:
- name: scTE
  role: Quantifies TE expression per cell from Cell Ranger BAMs, assigning reads first
    to gene exons then to TEs, and emits an .h5ad cell-by-TE matrix.
  repo: https://github.com/JiekaiLab/scTE
- name: scTE_build
  role: Builds the hg38 scTE index from GENCODE GTF and UCSC rmsk repeat annotations;
    also supports custom -te/-gene references.
  repo: https://github.com/JiekaiLab/scTE
- name: Cell Ranger
  role: Upstream 10x pipeline whose BAM output (CB:Z/UB:Z tags) is the required input
    format for scTE in this workflow.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21808-x
    title: Identifying transposable element expression dynamics and heterogeneity
      during development at the single-cell level with a processing pipeline scTE
  source_content_hash: sha256:4b8c62b8f59bb2b2b6b3e4d93b997657b44ff13290c1cec5370c1ee7d877dcf1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiekaiLab/scTE
  tools:
  - Cell Ranger
  - scTE
  - scTE_build
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# differential-te-expression-across-cardiac-lineage-branches

## Summary

Quantifies single-cell transposable element expression with scTE on 10x scRNA-seq BAMs (E-MTAB-6268 human cardiac differentiation) and contrasts TE expression between NKX2-5+ definitive cardiomyocyte and SPARC+ non-contractile branches, recovering branch-specific TE markers.

## When to use

Use when single-cell RNA-seq BAMs from a branching differentiation system need TE-level quantification and comparison of TE expression between annotated cell-fate branches (e.g., contracting vs non-contractile cardiomyocyte lineages).

## When NOT to use

- Input is already a gene or TE feature count table (e.g., a matrix from te_counter) rather than aligned BAM/SAM reads — scTE requires alignment files as input.
- Only bulk RNA-seq data are available; TE heterogeneity between single-cell branches cannot be assessed without single-cell barcoded libraries.
- The organism/genome is not among the assemblies scTE_build supports (mm10, hg38, panTro6, macFas5, dm6, danRer11, xenTro9) and no custom BED/GTF is available.

## Inputs

- Cell Ranger BAM files from human cardiac differentiation (E-MTAB-6268) with CB:Z and UB:Z barcode/UMI tags
- hg38 GENCODE gene annotation GTF
- UCSC hg38 rmsk.txt.gz repeat annotation

## Outputs

- Cell-by-TE count matrix in HDF5/.h5ad format (cardiac_te_expression.h5ad)
- LTR7 vs HERVH-int Pearson correlation estimate across cells (~0.8)
- Branch-level differential TE summary table (NKX2-5+ dCM vs SPARC+ non-contractile)

## How to apply

Build an scTE genome index with scTE_build for hg38, which pulls the GENCODE v30 GTF and UCSC hg38 rmsk.txt.gz repeat annotation. Run scTE on the Cell Ranger BAMs in default read-allocation mode (reads assigned first to gene exons, then to TEs), passing -CB CB -UMI UB and --hdf5 True since Cell Ranger stores barcodes/UMIs in CB:Z/UB:Z tags; budget roughly 10 GB memory per thread. Extract the cell-by-TE matrix from the .h5ad output and first validate the run by checking the Pearson correlation between LTR7 and HERVH-int across cells (~0.8) and co-downregulation of LTR7/HERVH-int with POU5F1 during differentiation. Then subset cells into NKX2-5+ and SPARC+ branches and test TE expression between contracting (dCM) and non-contractile cells, expecting LTR32, MER57A-int and MER45A enriched in dCMs and MLT1H1, HERVIP10B-int and LTR5A in non-contractile cells. Write the TE matrix to cardiac_te_expression.h5ad and the branch verification results to a summary table.

## Related tools

- **scTE** (Quantifies TE expression per cell from Cell Ranger BAMs, assigning reads first to gene exons then to TEs, and emits an .h5ad cell-by-TE matrix.) — https://github.com/JiekaiLab/scTE
- **scTE_build** (Builds the hg38 scTE index from GENCODE GTF and UCSC rmsk repeat annotations; also supports custom -te/-gene references.) — https://github.com/JiekaiLab/scTE
- **Cell Ranger** (Upstream 10x pipeline whose BAM output (CB:Z/UB:Z tags) is the required input format for scTE in this workflow.)

## Examples

```
scTE -i cardiac.bam -o cardiac_te -x hg38.exclusive.idx --hdf5 True -CB CB -UMI UB
```

## Evaluation signals

- LTR7–HERVH-int Pearson correlation across cells is approximately 0.8.
- LTR7 and HERVH-int expression co-downregulates with POU5F1 over the course of differentiation.
- LTR32, MER57A-int and MER45A show differential expression enriched in NKX2-5+ dCM cells.
- MLT1H1, HERVIP10B-int and LTR5A are specifically expressed in SPARC+ non-contractile cells.
- Output .h5ad contains a cell-by-TE matrix whose barcodes match the CB:Z tags of the input BAMs.

## Limitations

- Default read-allocation mode assigns reads overlapping exonic/UTR TE copies to the gene only, so exonic TE contributions are suppressed unless -m inclusive is set; -m nointron drops intronic TEs entirely.
- scTE's TE information content is lower than that of genes and serves mainly as an addendum to gene information.
- High memory footprint (~10 GB per thread) when running scTE on BAM input.

## Evidence

- [readme] Default exon-first allocation: "The scTE algorithm will allocate 
reads first to gene exons, and then to TEs by default."
- [readme] Cell Ranger tag requirement: "For `bam` file generated by [Cell Ranger](https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/what-is-cell-ranger) etc, the cell barcodes and UMI need to be"
- [methods] h5ad output and tag options: "$ scTE -i inp.bam -o out -x mm10.exclusive.idx --hdf5 True -CB CR -UMI UR"
- [discussion] LTR7/HERVH validation phenomenon: "By taking advantage of this tool, we could observe previously identified phenomena such as MERVL and LTR7/HERVH expression in mouse and human ESCs, respectively."
- [discussion] TEs as addendum to gene information: "Overall, whilst the information content of TEs is lower than that of genes, TEs are a useful addendum to the gene information"
