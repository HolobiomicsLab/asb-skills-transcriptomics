---
name: multimapping-read-te-subtype-collapsing
description: Use when single-cell RNA-seq BAM/SAM reads multi-map to transposable
  element subtypes and a cell-by-gene/TE count matrix must be generated without double-counting
  ambiguous reads across subtypes or overlapping genes.
when_to_use_negative:
- Input is already a gene/TE count matrix rather than an aligned BAM/SAM — no read
  allocation is needed
- Bulk RNA-seq (non-single-cell) data without cell barcodes and UMIs, where the per-cell
  demultiplexing logic does not apply
- Single-cell ATAC-seq chromatin accessibility quantification — use scTEATAC with
  its own index instead
edam_operation: http://edamontology.org/operation_3806
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_0080
tools:
- name: scTE
  role: Performs the exclusive/inclusive read allocation, multimapping TE subtype
    collapsing, barcode demultiplexing, and UMI collapsing on the BAM input
  repo: github.com/JiekaiLab/scTE
- name: scTE_build
  role: Builds the gene/TE genome index (from GTF and BED) that scTE uses for fast
    read-to-feature assignment via -x
  repo: github.com/JiekaiLab/scTE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21808-x
    title: Identifying transposable element expression dynamics and heterogeneity
      during development at the single-cell level with a processing pipeline scTE
  source_content_hash: sha256:74295c4ce2c8c855d63b9037253eac13888085ec0cd72011add47c2f84ee6372
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scTE
  - scTE_build
  edam_operation: http://edamontology.org/operation_3806
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# Multimapping read TE subtype collapsing

## Summary

scTE's read-allocation strategy for single-cell TE quantification: reads are assigned first to gene exons, then to TEs, with multimapping TE reads collapsed to a single TE subtype before UMI/barcode demultiplexing, yielding a cell-by-gene/TE count matrix.

## When to use

Use when single-cell RNA-seq BAM/SAM reads multi-map to transposable element subtypes and a cell-by-gene/TE count matrix must be generated without double-counting ambiguous reads across subtypes or overlapping genes.

## When NOT to use

- Input is already a gene/TE count matrix rather than an aligned BAM/SAM — no read allocation is needed
- Bulk RNA-seq (non-single-cell) data without cell barcodes and UMIs, where the per-cell demultiplexing logic does not apply
- Single-cell ATAC-seq chromatin accessibility quantification — use scTEATAC with its own index instead

## Inputs

- Unfiltered BAM/SAM alignment file from Cell Ranger or STARsolo with cell barcode (CB/CR) and UMI (UB/UR) tags
- Gene annotation GTF (e.g. GENCODE, Ensembl release-102/103)
- Transposable element annotation BED derived from UCSC rmsk (RepeatMasker) files
- scTE genome index built via scTE_build (-x)

## Outputs

- Cell-by-gene/TE count matrix (rows = cells, columns = gene/TE features)
- Output in CSV (default) or .h5ad format with --hdf5 True

## How to apply

First build the genome index with scTE_build from a gene GTF (e.g. GENCODE/Ensembl) and a TE BED (e.g. UCSC rmsk), producing the index passed via -x. Load an unfiltered Cell Ranger or STARsolo BAM/SAM with barcode tags (CB/CR) and UMI tags (UB/UR). By default scTE applies the exclusive hierarchy — reads map first to gene exons, so TEs inside exon/UTR regions contribute only to the gene — and multimapping TE reads are collapsed to a single TE subtype so each read contributes at most one count. Choose the allocation mode explicitly: -m inclusive assigns such reads to both TEs and genes, while -m nointron removes TEs inside gene introns. scTE demultiplexes reads by cell barcode and collapses UMIs per cell per feature, then writes a count matrix (CSV default, --hdf5 True for .h5ad).

## Related tools

- **scTE** (Performs the exclusive/inclusive read allocation, multimapping TE subtype collapsing, barcode demultiplexing, and UMI collapsing on the BAM input) — github.com/JiekaiLab/scTE
- **scTE_build** (Builds the gene/TE genome index (from GTF and BED) that scTE uses for fast read-to-feature assignment via -x) — github.com/JiekaiLab/scTE

## Examples

```
scTE -i inp.bam -o out -x mm10.exclusive.idx --hdf5 True -CB CR -UMI UR
```

## Evaluation signals

- Output count matrix exists with cells as rows and gene/TE features as columns, in the requested CSV or h5ad format
- Each multimapped TE read contributes to at most one TE subtype (collapsed), so total TE counts per cell do not exceed allocated reads
- In default exclusive mode, reads overlapping exon/UTR regions of GENCODE-annotated genes count toward the gene and not the TE score
- Switching to -m inclusive should increase TE counts for TEs inside gene exon/UTR regions relative to the exclusive run
- UMI collapsing yields at most one count per unique (cell barcode, UMI, feature) combination

## Limitations

- In default exclusive mode, TE reads falling inside exon/UTR regions of genes are lost to the TE score, undercounting those TEs unless -m inclusive is used
- TEs are highly repetitive; multimapping reads are collapsed to a single TE subtype, so subtype-level resolution of collapsed reads is inherently approximate
- Unfiltered alignment BAM/SAM is highly recommended as input; filtered files may lose reads assigned to empty droplets
- TE information content is lower than that of genes — TE counts serve mainly as an addendum to gene information

## Evidence

- [readme] Exclusive allocation hierarchy: "The scTE algorithm will allocate 
reads first to gene exons, and then to TEs by default."
- [readme] Inclusive mode decision point: "setting `–mode/-m inclusive` in scTE, which will instruct scTE to assign the reads to both TEs and genes"
- [readme] nointron mode decision point: "If you want to remove the TEs inside the intron 
of genes, you can sete `–mode/-m nointron` in scTE"
- [methods] Input requirement: "scTE makes BAM/SAM file as input, highly recommend to use unfiltered alignment file as input."
