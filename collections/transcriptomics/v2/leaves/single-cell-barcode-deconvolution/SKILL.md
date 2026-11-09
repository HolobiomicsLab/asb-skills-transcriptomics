---
name: single-cell-barcode-deconvolution
description: 'Use when starting a single-cell long-read analysis: you have packaged
  FASTQ files from PacBio or ONT runs plus a barcode-to-celltype assignment file (e.g.
  from Seurat on the paired 10x v2 short-read data), and need barcoded, cell-type-labeled
  reads.'
when_to_use_negative:
- Input is bulk long-read data with no paired single-cell (e.g. 10x v2) barcode-celltype
  assignments — barcodes cannot be mapped to cell types
- Reads lack embedded cellular barcodes/UMIs (e.g. standard Iso-Seq libraries) so
  deconvolution is meaningless
- You only need differential isoform testing on an already-aligned, already-assigned
  AllInfo table — use DiffSplicingAnalysis instead
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: scisorseqr
  role: R package providing GetBarcodes (barcode deconvolution from fastqs), STARalign/MMalign
    integration, and MapAndFilter for retaining high-confidence full-length spliced
    reads
  repo: https://github.com/noush-joglekar/scisorseqr
- name: STARlong
  role: Splice-aware alignment of the filtered barcoded long reads (PacBio) to the
    mm10 reference, writing BAMs to STARoutput
  repo: https://github.com/alexdobin/STAR/
- name: Minimap2
  role: Alternative splice-aware aligner used instead of STARlong, e.g. for Oxford
    Nanopore reads
  repo: https://github.com/lh3/minimap2
- name: Seurat
  role: Short-read single-cell clustering that produces the barcode-to-celltype assignment
    file used by GetBarcodes
- name: IsoQuant
  role: Sister package for downstream cell-type-specific isoform quantification once
    reads are barcoded and filtered
  repo: https://github.com/ablab/isoquant
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:79f1cc2e3dfe8453348e0db7539bd2685596d2e3c63250ea156e2eacdfe4e9ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noush-joglekar/scisorseqr
  tools:
  - IsoQuant
  - Minimap2
  - Python
  - R
  - STARlong
  - Seurat
  - bedtools
  - pandas
  - samtools
  - scisorseqr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-cell-barcode-deconvolution

## Summary

Deconvolves PacBio/ONT single-cell isoform sequencing (ScISOrSeq) reads by matching cellular barcodes extracted from long-read FASTQs to cell-type labels derived from short-read Seurat clustering, using the scisorseqr R package.

## When to use

Use when starting a single-cell long-read analysis: you have packaged FASTQ files from PacBio or ONT runs plus a barcode-to-celltype assignment file (e.g. from Seurat on the paired 10x v2 short-read data), and need barcoded, cell-type-labeled reads.

## When NOT to use

- Input is bulk long-read data with no paired single-cell (e.g. 10x v2) barcode-celltype assignments — barcodes cannot be mapped to cell types
- Reads lack embedded cellular barcodes/UMIs (e.g. standard Iso-Seq libraries) so deconvolution is meaningless
- You only need differential isoform testing on an already-aligned, already-assigned AllInfo table — use DiffSplicingAnalysis instead

## Inputs

- Packaged PacBio or ONT FASTQ files (e.g. fastq.gz per run) containing cellular barcodes and UMIs
- Barcode-to-celltype assignment file (BCClustAssignFile) derived from short-read single-cell Seurat analysis
- Reference genome FASTAs (per-chromosome, mm10) and GENCODE annotation GTF for subsequent filtering

## Outputs

- OutputFiltered/FilteredDeconvBC_AllFiles.csv — filtered per-read table with Barcode, UMI, Celltype and read identifiers
- Concatenated barcoded FASTQs in OutputFiltered ready for alignment
- Downstream LongReadInfo/AllInfo.gz per-read table (ReadName, Gene, Celltype, Barcode, UMI, IntronChain, ExonChain, Status)

## How to apply

Run scisorseqr::GetBarcodes on the FASTQ folder with the barcode-celltype assignment file (built from short-read Seurat clusters), setting chemistry = "v2", concatenate = TRUE, and filterReads = TRUE so only reads containing recognized barcodes are retained in OutputFiltered (FilteredDeconvBC CSV). This pairing step is what lets downstream STARlong/Minimap2-aligned reads be attributed to individual cell types; scisorseqr::STARalign then writes BAMs to STARoutput. Follow with MapAndFilter (filterFullLength = TRUE, GENCODE annotation GTF, per-chromosome FASTAs, cp_distance = 50, numThreads = 24, genomeVersion = "mm10") to keep full-length, spliced, poly-exonic reads. Rationale: only barcoded, high-confidence reads can feed per-cell-type isoform quantification with IsoQuant, so errors here propagate into the isoform matrix.

## Related tools

- **scisorseqr** (R package providing GetBarcodes (barcode deconvolution from fastqs), STARalign/MMalign integration, and MapAndFilter for retaining high-confidence full-length spliced reads) — https://github.com/noush-joglekar/scisorseqr
- **STARlong** (Splice-aware alignment of the filtered barcoded long reads (PacBio) to the mm10 reference, writing BAMs to STARoutput) — https://github.com/alexdobin/STAR/
- **Minimap2** (Alternative splice-aware aligner used instead of STARlong, e.g. for Oxford Nanopore reads) — https://github.com/lh3/minimap2
- **Seurat** (Short-read single-cell clustering that produces the barcode-to-celltype assignment file used by GetBarcodes)
- **IsoQuant** (Sister package for downstream cell-type-specific isoform quantification once reads are barcoded and filtered) — https://github.com/ablab/isoquant

## Examples

```
scisorseqr::GetBarcodes(fqFolder="fastqFolder", BCClustAssignFile="bc_celltype_assignments.csv", chemistry="v2", concatenate=TRUE, filterReads=TRUE)
```

## Evaluation signals

- OutputFiltered/FilteredDeconvBC_AllFiles.csv exists and every retained read has a non-empty Barcode and a Celltype assignment
- Reads without recognized barcodes (filterReads = TRUE) are excluded from the concatenated OutputFiltered FASTQs
- Downstream LongReadInfo/AllInfo.gz contains the expected columns (ReadName, Gene, Celltype, Barcode, UMI, IntronChain, ExonChain, Status)
- Aligned BAMs appear in STARoutput for every input FASTQ, confirming barcode retention through alignment
- Chemistry setting ("v2") matches the 10x chemistry of the paired short-read data, otherwise barcode whitelists mismatch and assignment rates collapse

## Limitations

- Package tested only on a CentOS x86_64 machine; alignment requires STARlong (PacBio) or Minimap2 (ONT) installations plus samtools, bedtools and Python >= 3.7
- Assignment quality depends on the paired short-read Seurat clustering; long-read clustering discrepancies may stem from sparsity of the isoform matrix due to lower long-read throughput

## Evidence

- [methods] GetBarcodes locates barcodes: "GetBarcodes(fqFolder = fastqFolder, BCClustAssignFile = bc_clust, 
            chemistry = "v2", concatenate = TRUE, filterReads = FALSE)"
- [methods] Step 1 of the workflow is barcode deconvolution from PacBio or ONT fastq output: "The first step is to deconvolve barcodes from either the PacBio or ONT fastq output"
- [methods] Barcode-celltype file comes from short-read Seurat analysis: "Assuming that short read single cell analysis has been performed using Seurat:"
- [readme] scisorseqr scope: deconvolution, alignment integration, mapping and filtering: [evidence span withheld — anchor status: external_doc]
- [readme] scisorseqr is a linux R-package for single-cell alternative splicing analysis: [evidence span withheld — anchor status: external_doc]
