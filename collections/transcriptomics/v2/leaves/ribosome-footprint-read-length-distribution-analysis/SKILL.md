---
name: ribosome-footprint-read-length-distribution-analysis
description: Use when aligned Ribo-seq footprints (e.g. length-split BAM files such
  as *.25-35.bam) must be validated as genuine ribosome-protected fragments before
  quantitative analysis or differential expression.
when_to_use_negative:
- Input is not ribosome profiling data (e.g. bulk RNA-seq footprints without RPF size
  constraints), since the 25-35 nt selection and P-site offset logic assume ribosome-protected
  fragments
- The goal is quantitative differential expression rather than qualitative footprint
  validation — DESeq2 normalization and contrast testing are handled by a separate
  RiboDoc module
- Alignment files are not yet produced (raw FastQ); trimming, rRNA depletion and Hisat2/Bowtie2
  alignment must precede this QC step
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0623
tools:
- name: RiboDoc
  role: Docker/Snakemake pipeline orchestrating read-length distribution QC, metagene
    analysis, CDS enrichment and QC report generation
  repo: https://github.com/equipeGST/RiboDoc
- name: riboWaltz
  role: Two-step P-site offset estimation per read length and trinucleotide periodicity/metaplot
    visualization
  repo: https://github.com/LabTranslationalArchitectomics/riboWaltz
- name: HTSeq
  role: htseq-count feature counting of reads overlapping CDS versus UTR features
    for the CDS enrichment panel
- name: TRiP
  role: Alternative to riboWaltz for offset prediction and periodicity graph creation,
    selected when the dataset does not align enough material
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:f72acb1fd366d6351c8b95606a18c466fac0e97adf04c8acdbb0128efdc2698c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/equipeGST/RiboDoc
  repo_url: https://github.com/equipeGST/RiboDoc
  tools:
  - BEDTools
  - HTSeq
  - RiboDoc
  - TRiP
  - riboWaltz
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# ribosome-footprint-read-length-distribution-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Qualitative quality-control module for ribosome profiling data that verifies RiboSeq reads correspond to active ribosome footprints by tabulating footprint counts per read length, estimating P-site offsets per length with riboWaltz/TRiP, and checking 3-nt periodicity and CDS enrichment.

## When to use

Use when aligned Ribo-seq footprints (e.g. length-split BAM files such as *.25-35.bam) must be validated as genuine ribosome-protected fragments before quantitative analysis or differential expression.

## When NOT to use

- Input is not ribosome profiling data (e.g. bulk RNA-seq footprints without RPF size constraints), since the 25-35 nt selection and P-site offset logic assume ribosome-protected fragments
- The goal is quantitative differential expression rather than qualitative footprint validation — DESeq2 normalization and contrast testing are handled by a separate RiboDoc module
- Alignment files are not yet produced (raw FastQ); trimming, rRNA depletion and Hisat2/Bowtie2 alignment must precede this QC step

## Inputs

- Length-split BAM or BED alignment files of Ribo-seq reads (e.g. *.25-35.bam for the 25-35 nt RPF range)
- GFF3 genome annotation with CDS, start and stop codon coordinates
- Reference genome/transcriptome sequences (e.g. Ensembl Saccharomyces_cerevisiae.R64-1-1 FASTA)

## Outputs

- Read-length distribution plot (counts of RPFs per length, modal length expected near 28 nt)
- Metagene periodicity plot around start (-50/+100 nt) and stop (-100/+50 nt) codons showing 3-nt phasing
- CDS overrepresentation/enrichment panel (footprints in CDS versus UTRs)
- Optional PCA and pairwise Spearman correlation matrix panels
- Consolidated QC report PDF (ribodoc_qc_report.pdf)

## How to apply

Load the length-split BAM/BED alignment files together with a GFF3 genome annotation. Tabulate footprint counts at each read length and plot the read-length distribution, checking that the modal length falls near 28 nt (RiboDoc's default RPF selection keeps reads 25-35 nt). Use riboWaltz's two-step P-site offset estimation per read length, pool all annotated CDS into a metagene, and compute footprint density around start (-50/+100 nt) and stop (-100/+50 nt) codons using the read 5' ends to display 3-nt periodicity. Count reads overlapping CDS versus UTR features with HTSeq-style feature counting and compute the CDS overrepresentation enrichment. Optionally add a PCA and pairwise Spearman correlation matrix for replicate reproducibility, and assemble distribution, periodicity, enrichment and statistical panels into a single QC report (ribodoc_qc_report.pdf).

## Related tools

- **RiboDoc** (Docker/Snakemake pipeline orchestrating read-length distribution QC, metagene analysis, CDS enrichment and QC report generation) — https://github.com/equipeGST/RiboDoc
- **riboWaltz** (Two-step P-site offset estimation per read length and trinucleotide periodicity/metaplot visualization) — https://github.com/LabTranslationalArchitectomics/riboWaltz
- **HTSeq** (htseq-count feature counting of reads overlapping CDS versus UTR features for the CDS enrichment panel)
- **TRiP** (Alternative to riboWaltz for offset prediction and periodicity graph creation, selected when the dataset does not align enough material)

## Examples

```
docker pull equipegst/ribodoc && ./ribodoc run --project ./my_project  # pipeline step: offset prediction and periodicity graphs via riboWaltz or TRiP
```

## Evaluation signals

- Modal read length in the distribution plot is ~28 nt (default RPF selection spans 25-35 nt)
- Metagene shows a clear 3-nt periodicity with CDS-anchored phasing in the -50/+100 nt (start) and -100/+50 nt (stop) windows
- Footprints are overrepresented within the CDS relative to UTRs in the feature-count enrichment panel
- P-site offsets vary smoothly and coherently across neighbouring read lengths (riboWaltz two-step algorithm maximizes offset coherence)
- Replicates cluster together in PCA and show high pairwise Spearman correlation coefficients

## Limitations

- Datasets that do not align enough material cannot use the riboWaltz pipeline; TRiP should be selected instead (as noted for the GEO GSE173856 example dataset)
- riboWaltz requires R >= 3.3.0 plus specific CRAN (data.table, ggplot2, ggrepel) and Bioconductor (GenomicAlignments, GenomicFeatures, GenomicRanges, IRanges, Biostrings) dependencies
- The QC confirms correspondence to active ribosomes but does not itself perform differential expression; low-quality samples must be removed and RiboDoc rerun for downstream analysis

## Evidence

- [readme] a two-step algorithm where offset information is passed through populations of reads with different length: [evidence span withheld — anchor status: external_doc]
