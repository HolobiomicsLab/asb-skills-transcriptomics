---
name: single-cell-long-read-splicing-analysis
description: Use when long-read FASTQs (PacBio/ONT) from single cells assigned to
  cell types must be converted into a cell-type-specific alternative splicing/DIE
  analysis, or when bulk DIE events must be traced to one vs. multiple cell types.
when_to_use_negative:
- 'Short-read-only data: splicing inference requires full-length spliced long reads,
  so short-read junction counts cannot substitute for the isoform-level test'
- The goal is gene differential expression rather than isoform/alternative-splicing
  differences between conditions or cell types
- 'Bulk tissue RNA-seq without single-cell assignment: cell-type tracing of DIE requires
  per-cell-type isoform usage (neuron/non-neuron counts), unavailable in bulk'
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: scisorseqr
  role: R-package orchestrating barcode deconvolution, alignment, filtering, quantification
    and differential isoform analysis from fastqs to cell-type-specific splicing results
  repo: https://github.com/noush-joglekar/scisorseqr
- name: STARlong
  role: Spliced aligner used via STARalign() for PacBio long reads against the reference
    genome
  repo: https://github.com/alexdobin/STAR
- name: Minimap2
  role: Alternative spliced aligner used via MMalign() for Oxford Nanopore (or PacBio)
    reads
  repo: https://github.com/lh3/minimap2
- name: IsoQuant
  role: Sister package for isoform/TSS/PolyA quantification from the AllInfo file
  repo: https://github.com/ablab/isoquant
- name: Seurat
  role: Short-read single-cell clustering in R producing the barcode-celltype assignment
    file consumed by scisorseqr
- name: samtools
  role: Alignment post-processing supporting MapAndFilter for full-length spliced
    reads
- name: bedtools
  role: Genomic interval operations supporting the mapping/filtering step
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:accca26f10d9894dc53112fb4f36e56b2d042352b88b27bc6c486149c406104e
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.72
n_claims: 25
---

# single-cell-long-read-splicing-analysis

## Summary

End-to-end scisorseqr pipeline for detecting differential isoform expression (DIE) between brain regions at single-cell resolution using single-cell long-read sequencing (ScISOrSeq), from FASTQ barcode deconvolution through cell-type-level DIE testing and tracing of bulk DIE events to constituent cell types.

## When to use

Use when long-read FASTQs (PacBio/ONT) from single cells assigned to cell types must be converted into a cell-type-specific alternative splicing/DIE analysis, or when bulk DIE events must be traced to one vs. multiple cell types.

## When NOT to use

- Short-read-only data: splicing inference requires full-length spliced long reads, so short-read junction counts cannot substitute for the isoform-level test
- The goal is gene differential expression rather than isoform/alternative-splicing differences between conditions or cell types
- Bulk tissue RNA-seq without single-cell assignment: cell-type tracing of DIE requires per-cell-type isoform usage (neuron/non-neuron counts), unavailable in bulk

## Inputs

- PacBio or ONT FASTQ files of single-cell long reads
- Barcode-cluster assignment file from short-read Seurat analysis (barcode-celltype file)
- Reference genome FASTA and annotation GTF (mm10, GENCODE gencode10 index)
- Gene-wise DIE results table (Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, FDR) from DiffSplicingAnalysis
- Neuron and non-neuron isoform count tables from the deposited isoformAtlas dataset

## Outputs

- Filtered full-length spliced read information files (LongReadInfo/AllInfo)
- Isoform, TSS, and PolyA quantifications per cell type
- DIE significance tables per pairwise cell-type/region comparison (TreeTraversal output)
- Per-gene cell-type model classification table (die_model_classification.csv) with model class, delta-PI(subtype)/delta-PI(bulk) ratios, and class summary statistics

## How to apply

Run the scisorseqr steps in order: GetBarcodes to deconvolve barcodes from fastq output using a Seurat-derived barcode-celltype file (chemistry='v2'); align with STARlong for PacBio reads or Minimap2 for ONT reads; MapAndFilter to retain full-length, spliced reads against the mm10/GENCODE reference; InfoPerLongRead to concatenate read information; then IsoQuant and DiffSplicingAnalysis with typeOfTest='Iso' (isoform tests outperform exon tests), minNumReads=25, groupingFactor='Celltype' (threshold=10 for quantification). To attribute bulk DIE between hippocampus and PFC to cell types, compute per-gene delta-PI in the bulk and in each neuron/non-neuron subtype, then classify each gene by the delta-PI(subtype)/delta-PI(bulk) ratio into Single-Cell-Type, Both-Cell-Type, or No-Cell-Type models and record the class fractions.

## Related tools

- **scisorseqr** (R-package orchestrating barcode deconvolution, alignment, filtering, quantification and differential isoform analysis from fastqs to cell-type-specific splicing results) — https://github.com/noush-joglekar/scisorseqr
- **STARlong** (Spliced aligner used via STARalign() for PacBio long reads against the reference genome) — https://github.com/alexdobin/STAR
- **Minimap2** (Alternative spliced aligner used via MMalign() for Oxford Nanopore (or PacBio) reads) — https://github.com/lh3/minimap2
- **IsoQuant** (Sister package for isoform/TSS/PolyA quantification from the AllInfo file) — https://github.com/ablab/isoquant
- **Seurat** (Short-read single-cell clustering in R producing the barcode-celltype assignment file consumed by scisorseqr)
- **samtools** (Alignment post-processing supporting MapAndFilter for full-length spliced reads)
- **bedtools** (Genomic interval operations supporting the mapping/filtering step)

## Examples

```
DiffSplicingAnalysis(configFile = config, typeOfTest = 'Iso', minNumReads = 25, is.hier = FALSE)
```

## Evaluation signals

- The die_model_classification.csv table must contain Gene and model-class fields with one row per DIE gene, matching the input DIE result count (e.g. 258 genes: 151/81/26 for Single-/Both-/No-Cell-Type models)
- Class fractions must fall in reported ranges: Single-Cell-Type 75% +/- 2.3 SE, Both ~24.4%, No ~0.3%
- DiffSplicingAnalysis must be run with typeOfTest='Iso' and minNumReads=25; isoform tests are expected to outperform exon tests for DIE
- Only full-length, spliced reads passing MapAndFilter should enter downstream quantification
- Bulk DIE between HIPP and PFC should reproduce 395 genes with DIE, partially involving isoforms novel to GENCODE

## Limitations

- The isoform matrix is sparse due to lower long-read throughput, causing reproducibility discrepancies in long-read clustering
- Generalizability of DIE patterns to disease case-control studies, adult brain regions, and other species is untested
- scisorseqr has only been tested on a CentOS x86_64 machine
- Traditional transcriptional cell-type definitions may not correspond to isoform expression programs and may need re-definition

## Evidence

- [methods] scisorseqr workflow: "The *scisorseqr* package allows you to go from short-read, single cell assignments and a set of fastq files to a full-blown cell-type specific analysis of alternative splicing patterns."
- [methods] DiffSplicingAnalysis invocation: "DiffSplicingAnalysis(configFile = config, typeOfTest = 'Iso', minNumReads = 25, is.hier = FALSE)"
- [methods] filtering rationale: [evidence span withheld — anchor status: not_found]
- [abstract] isoform vs exon test: "Isoform tests for DIE show better performance than exon tests."
- [abstract] primary finding: "Mostly, one cell type is responsible for brain-region specific DIE. However, for fewer genes, multiple cell types influence DIE."
- [readme] tool requirements: [evidence span withheld — anchor status: external_doc]
