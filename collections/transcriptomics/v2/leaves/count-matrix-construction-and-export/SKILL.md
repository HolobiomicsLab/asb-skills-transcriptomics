---
name: count-matrix-construction-and-export
description: Use when a scisorseqr pipeline has produced LongReadInfo/AllInfo and
  IsoQuantOutput files and you need a cell_type × isoform count matrix plus a per-read
  assignment table exported as CSV.
when_to_use_negative:
- Input is raw fastq.gz files that have not yet been barcode-deconvolved, aligned,
  and filtered — run GetBarcodes/STARalign/MMalign, MapAndFilter, and InfoPerLongRead
  first
- The goal is short-read gene-level quantification rather than cell-type-specific
  isoform/alternative-splicing counts from long reads
- Only exon-level usage is needed and IsoQuant outputs would suffice without matrix
  assembly
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: scisorseqr
  role: R-package providing IsoQuant to quantify isoforms, TSS, and PolyA sites per
    cell type and utilities to assemble the count matrix
  repo: https://github.com/noush-joglekar/scisorseqr
- name: Python/pandas
  role: Used to assemble and export the per-cell-type isoform count matrix as CSV
- name: R
  role: Language in which scisorseqr and the matrix assembly run
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:2e9272a9a4feab8e710dd48fb54124231123cda309587ff9bfeeddde2ef7f91c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noush-joglekar/scisorseqr
  tools:
  - Minimap2
  - Python
  - Python/pandas
  - R
  - STARlong
  - bedtools
  - pandas
  - samtools
  - scisorseqr
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# count-matrix-construction-and-export

## Summary

Assembles a per-cell-type isoform (and TSS/PolyA) count matrix from the scisorseqr IsoQuantOutput files and exports it as a CSV alongside the filtered per-read assignment table, so that cell-type-specific isoform abundance can be reused in downstream differential analyses.

## When to use

Use when a scisorseqr pipeline has produced LongReadInfo/AllInfo and IsoQuantOutput files and you need a cell_type × isoform count matrix plus a per-read assignment table exported as CSV.

## When NOT to use

- Input is raw fastq.gz files that have not yet been barcode-deconvolved, aligned, and filtered — run GetBarcodes/STARalign/MMalign, MapAndFilter, and InfoPerLongRead first
- The goal is short-read gene-level quantification rather than cell-type-specific isoform/alternative-splicing counts from long reads
- Only exon-level usage is needed and IsoQuant outputs would suffice without matrix assembly

## Inputs

- LongReadInfo/AllInfo.gz file from scisorseqr::InfoPerLongRead (per-read info: ReadName, Gene, Celltype, Barcode, UMI, IntronChain, ExonChain, Status)
- OutputFiltered/FilteredDeconvBC_AllFiles.csv of barcoded, filtered reads

## Outputs

- Per-cell-type isoform count matrix (cell_type, isoform/gene id, counts) exported as CSV
- IsoQuantOutput directory with per-cell-type isoform, TSS, and PolyA site quantifications
- Filtered per-read assignment table CSV with ReadName, Gene, Celltype, Barcode, UMI, IntronChain, ExonChain, Status columns

## How to apply

After scisorseqr::InfoPerLongRead has built LongReadInfo/AllInfo.gz (minTimesIsoObserve = 5, rmTmpFolder = TRUE), run scisorseqr::IsoQuant with AllInfoFile = 'LongReadInfo/AllInfo' and Iso = TRUE, TSS = TRUE, PolyA = TRUE to quantify isoforms, transcription-start-sites, and PolyA sites per cell type into IsoQuantOutput. Assemble the per-cell-type isoform count matrix from these files (e.g. via MakeMatrices), keeping cell_type, isoform/gene id, and counts fields. Export the matrix as a CSV together with the filtered per-read table from OutputFiltered / LongReadInfo, which carries ReadName, Gene, Celltype, Barcode, UMI, IntronChain, ExonChain, and Status columns. The rationale is that IsoQuant collapses read-level information into per-cell-type summaries, enabling downstream differential isoform expression testing (DiffSplicingAnalysis with FDR and ΔΠ cutoffs).

## Related tools

- **scisorseqr** (R-package providing IsoQuant to quantify isoforms, TSS, and PolyA sites per cell type and utilities to assemble the count matrix) — https://github.com/noush-joglekar/scisorseqr
- **Python/pandas** (Used to assemble and export the per-cell-type isoform count matrix as CSV)
- **R** (Language in which scisorseqr and the matrix assembly run)

## Examples

```
IsoQuant(AllInfoFile = 'LongReadInfo/AllInfo', Iso = TRUE, TSS = TRUE, PolyA = TRUE)
```

## Evaluation signals

- The pipeline script runs end-to-end without errors
- Both output files (count matrix CSV and per-read assignment table) exist
- The count matrix contains cell_type, isoform/gene id, and counts fields
- All counts in the matrix are non-negative integers
- The per-read table retains the expected columns: ReadName, Gene, Celltype, Barcode, UMI, IntronChain, ExonChain, Status

## Limitations

- The isoform matrix is sparse due to lower long-read throughput, which can cause discrepancies in downstream clustering
- IsoQuant per-cell-type quantification depends on minTimesIsoObserve = 5, so isoforms observed in fewer reads are excluded
- A brain-wide map of isoform expression at single-cell level is not provided; coverage is limited to HIPP and PFC at postnatal day 7

## Evidence

- [methods] IsoQuant quantifies isoforms/TSS/PolyA per cell type: "IsoQuant(AllInfoFile = 'LongReadInfo/AllInfo',
          Iso = TRUE, TSS = TRUE, PolyA = TRUE)"
- [methods] InfoPerLongRead concatenates per-read information feeding the count matrix: "InfoPerLongRead(barcodeOutputFile = 
                  'OutputFiltered/FilteredDeconvBC_AllFiles.csv',"
- [readme] scisorseqr purpose spans barcode deconvolution to differential analysis: [evidence span withheld — anchor status: external_doc]
- [discussion] Sparse isoform matrix limits downstream analyses: "which could be partially due to the sparsity of the isoform matrix due to lower long-read throughput"
