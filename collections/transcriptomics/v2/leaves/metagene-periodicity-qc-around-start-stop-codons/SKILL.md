---
name: metagene-periodicity-qc-around-start-stop-codons
description: Use when length-selected Ribo-seq alignments (e.g. 25-35 nt RPF BAM/BED
  files) and a GFF3 annotation are available and you must verify footprints reflect
  active ribosomes before differential analysis.
when_to_use_negative:
- The input dataset does not align enough material for the riboWaltz pipeline — in
  that case RiboDoc should be run with TRiP selected instead (as noted for the GEO
  GSE173856 example dataset)
- The goal is differential expression or quantification rather than qualitative footprint
  validation — use the DESeq2 step of RiboDoc instead
- Input is raw FastQ; adapter trimming, rRNA/tRNA contaminant removal with Bowtie2,
  and alignment with HISAT2/Bowtie2 must precede this QC
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0623
tools:
- name: RiboDoc
  role: Docker/Snakemake pipeline orchestrating the metagene periodicity QC and assembling
    the QC report
  repo: https://github.com/equipeGST/RiboDoc
- name: riboWaltz
  role: Computes optimal P-site offsets per read length and generates metagene/trinucleotide-periodity
    plots pooled over annotated CDS
  repo: https://github.com/LabTranslationalArchitectomics/riboWaltz
- name: TRiP
  role: Alternative to riboWaltz for offset prediction and periodicity graph creation
    when the dataset is too small for riboWaltz
- name: HTSeq
  role: Feature counting of reads overlapping CDS versus UTR to compute CDS overrepresentation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:e00058cca6af03bb164d5dcc4c393f90c47b3e3f9ea881f756f1205d0e0f19f2
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
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Metagene periodicity QC around start/stop codons (riboWaltz/TRiP)

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A qualitative quality-control module of the RiboDoc pipeline that pools all annotated CDS into a metagene and verifies that ribosome footprints show 3-nt periodicity and CDS-anchored phasing, confirming the data correspond to active ribosome-protected fragments.

## When to use

Use when length-selected Ribo-seq alignments (e.g. 25-35 nt RPF BAM/BED files) and a GFF3 annotation are available and you must verify footprints reflect active ribosomes before differential analysis.

## When NOT to use

- The input dataset does not align enough material for the riboWaltz pipeline — in that case RiboDoc should be run with TRiP selected instead (as noted for the GEO GSE173856 example dataset)
- The goal is differential expression or quantification rather than qualitative footprint validation — use the DESeq2 step of RiboDoc instead
- Input is raw FastQ; adapter trimming, rRNA/tRNA contaminant removal with Bowtie2, and alignment with HISAT2/Bowtie2 must precede this QC

## Inputs

- Length-split BAM/BED alignment files (e.g. *.25-35.bam RPF selections)
- GFF3 genome annotation (e.g. from Ensembl)
- Reference genome FASTA (e.g. Saccharomyces_cerevisiae.R64-1-1.dna.toplevel.fa.gz)

## Outputs

- Metagene periodicity plot (footprint density around start −50/+100 and stop −100/+50 codons)
- Read-length distribution plot (modal length ~28 nt)
- CDS overrepresentation/enrichment panel
- QC report assembled as ribodoc_qc_report.pdf

## How to apply

Load the length-split BAM/BED alignment files together with the GFF3 genome annotation, and use riboWaltz-style P-site offset estimation per read length (riboWaltz or TRiP, as chosen in RiboDoc). Pool all annotated CDS into a metagene and compute footprint density using the 5′ end of each read in a user-definable window (default −50/+100 nt around the start codon and −100/+50 nt around the stop codon). Inspect the resulting metaplot for clear 3-nt periodicity starting ~12 nt upstream of the start codon and ending ~15 nt before the stop codon for ~28 nt RPFs. Pair this periodicity panel with the read-length distribution (modal length near 28 nt) and CDS-overrepresentation counting (HTSeq-style feature counting of CDS vs UTR reads) as the three independent qualitative QC checks; optionally add PCA and pairwise Spearman correlation for replicate reproducibility.

## Related tools

- **RiboDoc** (Docker/Snakemake pipeline orchestrating the metagene periodicity QC and assembling the QC report) — https://github.com/equipeGST/RiboDoc
- **riboWaltz** (Computes optimal P-site offsets per read length and generates metagene/trinucleotide-periodity plots pooled over annotated CDS) — https://github.com/LabTranslationalArchitectomics/riboWaltz
- **TRiP** (Alternative to riboWaltz for offset prediction and periodicity graph creation when the dataset is too small for riboWaltz)
- **HTSeq** (Feature counting of reads overlapping CDS versus UTR to compute CDS overrepresentation)

## Examples

```
docker pull equipegst/ribodoc
```

## Evaluation signals

- Modal read length in the distribution plot falls near 28 nt
- Metagene shows a clear 3-nt periodicity with CDS-anchored phasing for ~28 nt RPFs, starting 12 nt before the start codon and ending 15 nt before the stop codon
- Footprints are overrepresented within the CDS relative to UTR features
- All three panels (distribution, periodicity, enrichment) are assembled into a single ribodoc_qc_report.pdf
- P-site offsets are estimated per read length by the riboWaltz two-step algorithm passing offset information across read-length populations

## Limitations

- Datasets that do not align enough reads cannot support the riboWaltz pipeline and must be analyzed with TRiP instead
- QC validates only qualitative properties (periodicity, length distribution, CDS enrichment); replicate reproducibility requires the optional PCA and pairwise Spearman correlation panels
- No version number is stated for RiboDoc or its Docker package; no changelog exists for the repository

## Evidence
