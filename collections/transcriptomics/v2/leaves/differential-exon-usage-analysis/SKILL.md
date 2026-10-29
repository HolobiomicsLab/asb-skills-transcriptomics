---
name: differential-exon-usage-analysis
description: Use when long-read (PacBio/ONT) FASTQs have short-read cell-type assignments
  and you need per-exon PSI quantification and HIPP-vs-PFC delta-PSI tests, or single-cell-vs-spatial
  ΔΨ concordance.
when_to_use_negative:
- When transcript-level differential isoform expression testing is the goal and sufficient
  — the article reports 'Isoform tests for DIE show better performance than exon tests',
  so isoform tests are preferred
- When reads lack cell-type or spatial barcode assignments — scisorseqr's exon quantification
  depends on a barcode-celltype file from short-read (Seurat) analysis
- When the question is gene-level differential expression rather than alternative
  splicing or exon inclusion changes
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scisorseqr
  role: R-package executing barcode deconvolution, alignment, filtering, exon quantification
    (ExonQuant) and differential splicing tests (DiffSplicingAnalysis)
  repo: github.com/noush-joglekar/scisorseqr
- name: STARlong
  role: Spliced alignment of PacBio long reads to the mm10/gencode10 reference prior
    to exon quantification
  repo: https://github.com/alexdobin/STAR/
- name: Minimap2
  role: Spliced alignment of ONT reads (used for spatial long-read alignments and
    as alternative to STARlong)
  repo: https://github.com/lh3/minimap2
- name: pandas
  role: Python library used for computing per-exon delta-PSI and the single-cell-vs-spatial
    concordance statistics
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:6d8b2009782aee0a71ef86f323cbabf649a80beeabfe7c36379c771ab7fcf4fe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Minimap2
  - R
  - STARlong
  - pandas
  - scisorseqr
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# differential-exon-usage-analysis

## Summary

Quantifies per-exon inclusion (PSI) from cell-type-assigned long reads and tests for differential exon usage between hippocampus (HIPP) and prefrontal cortex (PFC) at P7, including reproduction of single-cell delta-PSI in spatial long-read data.

## When to use

Use when long-read (PacBio/ONT) FASTQs have short-read cell-type assignments and you need per-exon PSI quantification and HIPP-vs-PFC delta-PSI tests, or single-cell-vs-spatial ΔΨ concordance.

## When NOT to use

- When transcript-level differential isoform expression testing is the goal and sufficient — the article reports 'Isoform tests for DIE show better performance than exon tests', so isoform tests are preferred
- When reads lack cell-type or spatial barcode assignments — scisorseqr's exon quantification depends on a barcode-celltype file from short-read (Seurat) analysis
- When the question is gene-level differential expression rather than alternative splicing or exon inclusion changes

## Inputs

- PacBio or ONT FASTQ files of long reads from HIPP and PFC (P7 mouse)
- Short-read Seurat-derived barcode-celltype assignment file
- GENCODE (gencode10) GTF annotation and mm10 reference genome/STAR index
- scisorseqr concatenated per-read info file (LongReadInfo/AllInfo)
- Deposited 10X Visium spatial counts with ONT spatial long-read alignments

## Outputs

- Per-exon PSI quantifications grouped by cell type (ExonQuant output)
- Differential exon/isoform usage test results for HIPP vs PFC (e.g. 395 DIE genes reported)
- Per-exon concordance table exon_delta_psi_concordance.csv (exon ID, single-cell dPSI, spatial dPSI, same-direction flag)

## How to apply

Run the scisorseqr pipeline: GetBarcodes on the FASTQ output, align with STARlong (PacBio) or Minimap2 (ONT), MapAndFilter for full-length spliced reads, then InfoPerLongRead to concatenate read information. Quantify exon inclusion with ExonQuant(AllInfoFile='LongReadInfo/AllInfo', groupingFactor='Celltype', threshold=10), requiring ≥10 reads per group, then test region-specific exon usage with DiffSplicingAnalysis(typeOfTest='Iso', minNumReads=25, is.hier=FALSE). For spatial validation, compute per-exon spatial delta-PSI (HIPP minus PFC) from 10X Visium counts with ONT spatial long-read alignments for the same 40 region-specific exons, correlate single-cell and spatial ΔΨ vectors via Pearson r², count same-direction changes, and assess significance with a two-sided Bernoulli test. Note the article's caveat that isoform tests for DIE outperform exon tests, so exon-level results should be interpreted with that hierarchy in mind.

## Related tools

- **scisorseqr** (R-package executing barcode deconvolution, alignment, filtering, exon quantification (ExonQuant) and differential splicing tests (DiffSplicingAnalysis)) — github.com/noush-joglekar/scisorseqr
- **STARlong** (Spliced alignment of PacBio long reads to the mm10/gencode10 reference prior to exon quantification) — https://github.com/alexdobin/STAR/
- **Minimap2** (Spliced alignment of ONT reads (used for spatial long-read alignments and as alternative to STARlong)) — https://github.com/lh3/minimap2
- **pandas** (Python library used for computing per-exon delta-PSI and the single-cell-vs-spatial concordance statistics)

## Examples

```
DiffSplicingAnalysis(configFile=config, typeOfTest='Iso', minNumReads=25, is.hier=FALSE)
```

## Evaluation signals

- Concordance table exon_delta_psi_concordance.csv contains exactly 40 rows (one per region-specific exon)
- Same-direction fraction between single-cell and spatial ΔΨ is ~0.85 with two-sided Bernoulli p ≤ 3.5e-06
- Pearson r² between single-cell and spatial delta-PSI vectors falls in [0.5, 0.7] (reported 0.6)
- ExonQuant used groupingFactor='Celltype' with threshold=10 reads per group; DiffSplicingAnalysis used minNumReads=25
- Downstream DIE result reproduces the article's scale: 395 genes with DIE between hippocampus and PFC

## Limitations

- Isoform tests for DIE show better performance than exon tests, so exon-level calls are the lower-sensitivity option
- Sparsity of the isoform/exon matrix due to lower long-read throughput can cause discrepancies in cell-type-level comparisons
- Generalizability of observed region-specific splicing to adult brain regions, disease case-control studies, and species is untested

## Evidence

- [methods] ExonQuant with celltype grouping and 10-read threshold: "ExonQuant(AllInfoFile = 'LongReadInfo/AllInfo', 
          groupingFactor = "Celltype", threshold = 10)"
- [methods] DiffSplicingAnalysis invocation: "DiffSplicingAnalysis(configFile = config, 
                     typeOfTest = 'Iso', 
                     minNumReads = 25, is.hier = FALSE)"
- [abstract] Isoform tests outperform exon tests: "Isoform tests for DIE show better performance than exon tests."
- [discussion] 395 DIE genes between HIPP and PFC: "Region-specific isoform testing revealed 395 genes with DIE between hippocampus and PFC, partially caused by isoforms novel to GENCODE."
- [readme] scisorseqr purpose and spatial applicability: "The tools are also applicable to long-read spatial transcriptomics, and
can be used to resolve exon expression at the
[spatial](https://isoformatlas.com/) level"
