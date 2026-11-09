---
name: conversion-efficiency-estimation-from-spike-ins
description: Use when benchmarking sensitivity of a droplet-based 3' mRNA counting
  system with ERCC spike-ins, or when validating that detected UMI counts track expected
  molecule inputs.
when_to_use_negative:
- Input data are single-cell RNA-seq of real cells without synthetic spike-ins, so
  no expected molecule counts exist to correlate against
- Goal is cell capture or encapsulation efficiency, which this article measured separately
  (~50% cell capture rate) rather than via spike-ins
- Input is already a UMI count matrix — the pipeline is required only when counts
  must be re-derived from FASTQ
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: STAR
  role: Aligns Read1 (the cDNA insert) of ERCC runs to a STAR index built from the
    ERCC reference transcript sequences
  repo: https://github.com/10XGenomics/single-cell-3prime-paper
- name: Cell Ranger Single-Cell Software Suite
  role: Reference pipeline for sample demultiplexing, barcode processing and single-cell
    3′ gene counting whose UMI validity rules the ERCC counting follows
  repo: https://github.com/10XGenomics/cellranger
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:948c611ef89b2260a7b2b925a90d7f8f52c8eb0981bfebcc52ccd40884c8ac56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/single-cell-3prime-paper
  tools:
  - Cell Ranger Single-Cell Software Suite
  - STAR
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# conversion-efficiency-estimation-from-spike-ins

## Summary

Quantifies how efficiently a droplet-based scRNA-seq system converts known inputs of ERCC synthetic RNA molecules into detected UMI counts, and measures the correlation between sequenced UMI counts and expected ERCC molecule counts loaded into GEMs. This serves as an external, molecule-count-absolute benchmark of system sensitivity.

## When to use

Use when benchmarking sensitivity of a droplet-based 3' mRNA counting system with ERCC spike-ins, or when validating that detected UMI counts track expected molecule inputs.

## When NOT to use

- Input data are single-cell RNA-seq of real cells without synthetic spike-ins, so no expected molecule counts exist to correlate against
- Goal is cell capture or encapsulation efficiency, which this article measured separately (~50% cell capture rate) rather than via spike-ins
- Input is already a UMI count matrix — the pipeline is required only when counts must be re-derived from FASTQ

## Inputs

- ERCC Mix1 and Mix2 (1:50 dilution) GEM sequencing FASTQ files from SRA SRP073767 or support.10xgenomics.com/single-cell/datasets
- ERCC reference transcript sequences (cms_095047.txt from ThermoFisher)
- Expected ERCC molecule counts loaded into GEMs at the given dilution

## Outputs

- Inferred cDNA conversion efficiency per ERCC experiment (fraction of expected molecules observed as UMIs)
- Pearson correlation between mean UMI counts and expected ERCC molecule counts
- Coefficient of variation squared (CV2) across ERCC experiments
- Summary metrics table (e.g. ercc_sensitivity_metrics.csv)

## How to apply

Download the ERCC Mix1 and Mix2 (1:50 dilution) GEM sequencing FASTQ files from SRA accession SRP073767 (also mirrored at support.10xgenomics.com/single-cell/datasets) and the ERCC reference sequences from tools.thermofisher.com/content/sfs/manuals/cms_095047.txt. Build a STAR index on the ERCC reference and align Read1, which contains the cDNA insert, with STAR. Count valid cell-barcode/UMI combinations per ERCC transcript using the paper's validity rules: UMIs with sequencing quality score >10 are valid only if they are not homopolymers, and a UMI 1-Hamming-distance from another UMI (with more reads) for the same cell barcode is corrected to the more abundant UMI. Compute conversion efficiency per ERCC experiment as the fraction of expected ERCC molecules represented by counted UMIs, then compute the Pearson correlation between mean UMI counts and expected molecule counts and the CV2 across ERCC experiments.

## Related tools

- **STAR** (Aligns Read1 (the cDNA insert) of ERCC runs to a STAR index built from the ERCC reference transcript sequences) — https://github.com/10XGenomics/single-cell-3prime-paper
- **Cell Ranger Single-Cell Software Suite** (Reference pipeline for sample demultiplexing, barcode processing and single-cell 3′ gene counting whose UMI validity rules the ERCC counting follows) — https://github.com/10XGenomics/cellranger

## Examples

```
STAR --genomeDir ercc_index --readFilesIn ercc_R1.fastq.gz --readFilesCommand zcat --outSAMtype BAM SortedByCoordinate --outSAMmapqUnique 255
```

## Evaluation signals

- Inferred conversion efficiency for ERCC Spike-in Mix1 and Mix2 at 1:50 dilution falls within the reported 6.7–8.1% range
- Pearson correlation between mean UMI counts and expected ERCC molecule counts is ~0.96
- CV2 across all ERCC experiments is ~7%
- UMI counts only include UMIs with sequencing quality score >10 that are not homopolymers
- 1-Hamming-distance UMI correction applied per cell barcode and gene, resolving to the more-read UMI

## Limitations

- Conversion efficiency is inferred from loaded ERCC molecules, so pipetting/loading errors propagate into the efficiency estimate
- Earlier libraries used a 5 nt UMI and 5 bp Read2 instead of the standard configuration, so protocol heterogeneity across libraries must be accounted for
- Some sensitivity requires at least 1% of the expected recovered cell count as the barcode-order-of-magnitude anchor, so very low-cell ERCC runs may be poorly supported

## Evidence

- [discussion] UMI validity and homopolymer filter: "UMIs with sequencing quality score >10 were considered valid if they were not homopolymers."
- [discussion] UMI correction rule: "A UMI that is 1-Hamming-distance away from another UMI (with more reads) for the same cell barcode and gene is corrected to the UMI with more reads."
- [discussion] Read1 alignment to ERCC reference with STAR: "Read1, which contains the cDNA insert, was aligned to an appropriate reference genome using STAR35."
- [discussion] ERCC reference source: "For ERCC samples, ERCC reference (https://tools.thermofisher.com/content/sfs/manuals/cms_095047.txt) was used."
- [readme] Data availability of ERCC GEM runs: [evidence span withheld — anchor status: not_found]
