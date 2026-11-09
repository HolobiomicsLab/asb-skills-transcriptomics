---
name: ercc-spike-in-sensitivity-analysis
description: 'Use when benchmarking droplet scRNA-seq conversion efficiency or linearity
  using ERCC spike-ins: ERCC FASTQ from SRA SRP073767 is available and conversion
  efficiency, UMI-vs-molecule Pearson r, and CV2 must be recomputed.'
when_to_use_negative:
- Input data are whole-transcriptome cell FASTQs needing genome alignment (hg19/mm10),
  not ERCC-only spike-in benchmarking runs.
- The goal is gene expression quantification for biological samples (e.g., 68k PBMC
  profiling) rather than instrument sensitivity/linearity validation with synthetic
  RNAs.
- A gene-barcode UMI count matrix already exists for the ERCC runs and only downstream
  analysis, not UMI validity filtering and counting, is required.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: STAR
  role: Aligns Read1 (cDNA insert) of ERCC GEM runs to the ERCC-only reference index
- name: Cell Ranger Single-Cell Software Suite
  role: Reference implementation for sample demultiplexing, barcode processing, and
    3' gene/UMI counting whose UMI validity and correction rules are reimplemented
    for ERCC counting
  repo: https://github.com/10XGenomics/cellranger
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:1f1de953e27c1bcbed3bb5275555c6c6aae47f6d9903866501fa431e8bb486ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - STAR
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# ercc-spike-in-sensitivity-analysis

## Summary

Quantifies how efficiently the 10x Genomics droplet (GemCode) scRNA-seq system converts ERCC synthetic spike-in RNA molecules into detected UMI counts, by aligning Read1 cDNA inserts to an ERCC-only reference and comparing UMI counts to expected molecule counts.

## When to use

Use when benchmarking droplet scRNA-seq conversion efficiency or linearity using ERCC spike-ins: ERCC FASTQ from SRA SRP073767 is available and conversion efficiency, UMI-vs-molecule Pearson r, and CV2 must be recomputed.

## When NOT to use

- Input data are whole-transcriptome cell FASTQs needing genome alignment (hg19/mm10), not ERCC-only spike-in benchmarking runs.
- The goal is gene expression quantification for biological samples (e.g., 68k PBMC profiling) rather than instrument sensitivity/linearity validation with synthetic RNAs.
- A gene-barcode UMI count matrix already exists for the ERCC runs and only downstream analysis, not UMI validity filtering and counting, is required.

## Inputs

- ERCC Mix1 and Mix2 (1:50 dilution) GEM sequencing FASTQ files (SRA SRP073767 or 10x Genomics datasets site)
- ERCC reference transcript sequences (tools.thermofisher.com/content/sfs/manuals/cms_095047.txt)
- Expected ERCC molecule counts loaded into GEMs

## Outputs

- Inferred cDNA conversion efficiency per ERCC experiment (fraction of expected molecules represented by counted UMIs)
- Pearson correlation (r) between mean UMI counts and expected ERCC molecule counts
- Coefficient of variation squared (CV2) across ERCC experiments
- ercc_sensitivity_metrics.csv with recomputed metrics

## How to apply

Download the ERCC Mix1/Mix2 (1:50 dilution) GEM FASTQ files from SRA SRP073767 (mirrored at support.10xgenomics.com/single-cell/datasets) and the ERCC reference FASTA from tools.thermofisher.com/content/sfs/manuals/cms_095047.txt, then build a STAR index on the ERCC reference. Align Read1 (which contains the cDNA insert) with STAR, then count valid cell-barcode/UMI combinations per ERCC transcript using the paper's validity rules: UMIs with sequencing quality score >10 that are not homopolymers, with 1-Hamming-distance UMIs corrected to the UMI with more reads for the same cell barcode and gene. Compute inferred conversion efficiency per ERCC experiment as the fraction of expected ERCC molecules represented by counted UMIs, the Pearson correlation between mean UMI counts and expected molecule counts, and CV2 across ERCC experiments. Write the recomputed metrics to ercc_sensitivity_metrics.csv.

## Related tools

- **STAR** (Aligns Read1 (cDNA insert) of ERCC GEM runs to the ERCC-only reference index)
- **Cell Ranger Single-Cell Software Suite** (Reference implementation for sample demultiplexing, barcode processing, and 3' gene/UMI counting whose UMI validity and correction rules are reimplemented for ERCC counting) — https://github.com/10XGenomics/cellranger

## Examples

```
STAR --genomeDir ercc_index --readFilesIn ercc_mix1_R1.fastq.gz --readFilesCommand zcat --outSAMtype BAM SortedByCoordinate --outFileNamePrefix ercc_mix1_
```

## Evaluation signals

- Inferred conversion efficiency per ERCC experiment falls within the reported 6.7–8.1% range for both Mix1 and Mix2 at 1:50 dilution.
- Pearson r between mean sequenced UMI counts and expected ERCC molecule counts is approximately 0.96.
- CV2 across all ERCC experiments is approximately 7%.
- Every counted UMI passes validity checks: quality score >10, non-homopolymer, and 1-Hamming-distance correction applied toward the more-abundant UMI for the same cell barcode and gene.
- Output ercc_sensitivity_metrics.csv contains conversion efficiency, Pearson r, and CV2 columns with values matching the above ranges.

## Limitations

- Earlier libraries used a 5 nt UMI and 5 bp Read2 instead of the standard UMI, introducing protocol heterogeneity across libraries that must be handled when mixing runs.
- Cell-count estimates carry a 15–20% error, which can propagate into conversion-efficiency calculations that depend on expected molecule counts.
- Reported metrics (6.7–8.1% efficiency, r≈0.96, CV2≈7%) are specific to ERCC Spike-in Mix1/Mix2 at 1:50 dilution in the GemCode system and may not transfer to other concentrations or chemistries.

## Evidence

- [discussion] STAR alignment of ERCC Read1: "Read1, which contains the cDNA insert, was aligned to an appropriate reference genome using STAR35."
- [discussion] UMI validity rule: "UMIs with sequencing quality score >10 were considered valid if they were not homopolymers."
- [discussion] UMI correction rule: "A UMI that is 1-Hamming-distance away from another UMI (with more reads) for the same cell barcode and gene is corrected to the UMI with more reads."
- [discussion] ERCC reference source: "For ERCC samples, ERCC reference (https://tools.thermofisher.com/content/sfs/manuals/cms_095047.txt) was used."
- [readme] Data availability mirror: [evidence span withheld — anchor status: not_found]
