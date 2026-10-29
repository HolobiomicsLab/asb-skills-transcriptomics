---
name: long-read-transcriptome-isoform-detection
description: Use when analyzing long-read RNA-seq (nanopore or PacBio Iso-Seq) BAM/GFF3/BED6
  alignments of selected genes or spike-ins (e.g., SIRVs) to identify and quantify
  known or novel isoforms and tune detection thresholds.
when_to_use_negative:
- Input is raw FASTQ with no reference index and no intention to use the GMAP alignment
  panel — align reads first (GMAP or Minimap2) before isoform detection
- The goal is whole-transcriptome de-novo discovery across many samples — VIsoQLR
  is designed for close inspection and fine-tuning of one gene at a time in low-throughput
  splicing assays
- Isoform abundances are very small (e.g., ~1.7–4%), where LRS may overrepresent them
  and they are hardly distinguishable from noise in electropherograms
edam_operation: http://edamontology.org/operation_0250
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: VIsoQLR
  role: Interactive analyzer, viewer and editor for isoform identification, quantification,
    and CEC read-threshold tuning from long-read alignments
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: GMAP
  role: Aligner used to map raw FASTQ long reads to a reference index, producing the
    BAM uploaded to VIsoQLR
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: Minimap2
  role: Alternative aligner for long reads against a FASTA reference when GMAP is
    not used
  repo: https://github.com/lh3/minimap2
- name: TBLabFJD/VIsoQLR_benchmark
  role: Benchmark scripts including isoform_analysis_visoqlr_perc.R for comparing
    VIsoQLR results across read threshold values
  repo: https://github.com/TBLabFJD/VIsoQLR_benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:d500020967eb7c2bc4de785d91935896a6465b01861376ac28236f8d340886cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - GMAP
  - Minimap2
  - TBLabFJD/VIsoQLR_benchmark
  - VIsoQLR
  edam_operation: http://edamontology.org/operation_0250
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# long-read-transcriptome-isoform-detection

## Summary

Detect, quantify, and fine-tune isoforms from long-read sequencing data using VIsoQLR, an interactive Docker-based analyzer that defines consensus splice sites per gene and quantifies isoforms, with tunable CEC read thresholds for sensitivity analysis.

## When to use

Use when analyzing long-read RNA-seq (nanopore or PacBio Iso-Seq) BAM/GFF3/BED6 alignments of selected genes or spike-ins (e.g., SIRVs) to identify and quantify known or novel isoforms and tune detection thresholds.

## When NOT to use

- Input is raw FASTQ with no reference index and no intention to use the GMAP alignment panel — align reads first (GMAP or Minimap2) before isoform detection
- The goal is whole-transcriptome de-novo discovery across many samples — VIsoQLR is designed for close inspection and fine-tuning of one gene at a time in low-throughput splicing assays
- Isoform abundances are very small (e.g., ~1.7–4%), where LRS may overrepresent them and they are hardly distinguishable from noise in electropherograms

## Inputs

- GMAP-aligned BAM file (plus BAI index) of long reads for selected genes (e.g., SIRV spike-in alignments)
- GFF3 or BED6 alignment files as alternative inputs
- FASTQ raw reads plus a reference sequence (FASTA) if alignment via the built-in GMAP panel is needed
- Known isoform references (e.g., SIRV annotation of 68 transcripts) for comparison

## Outputs

- Detected isoform set with exon coordinates per analyzed gene
- Isoform abundance/quantification profile per gene
- Threshold-sweep results (isoform counts and cosine similarities, e.g., sirv_threshold_sweep_results.csv)

## How to apply

Pull the VIsoQLR Docker image (docker pull tblabfjd/visoqlr:latest) and run it with port mapping 8888:8888, then open the web interface. Upload GMAP-aligned reads in BAM (or GFF3/BED6) format; VIsoQLR automatically defines consensus exon coordinates (CECs) per gene and quantifies isoforms. In the 'Exon coordinates: Automatic detection' panel, set the 'Read threshold (%)' parameter, which selects candidate CECs based on the proportion of exon coordinates among all reads (default 2%); lowering it detects more low-abundance isoforms at the cost of precision. Optionally pad candidates, merge close splice sites within a given distance, and import known isoforms as references for comparison. For sensitivity analysis, sweep the threshold (e.g., 0.25, 0.5, 1, 2, 3 percent on SIRVs, yielding 65, 63, 59, 49, 41 of 68 isoforms) and score quantification accuracy via cosine similarity against a reference isoform profile.

## Related tools

- **VIsoQLR** (Interactive analyzer, viewer and editor for isoform identification, quantification, and CEC read-threshold tuning from long-read alignments) — https://github.com/TBLabFJD/VIsoQLR
- **GMAP** (Aligner used to map raw FASTQ long reads to a reference index, producing the BAM uploaded to VIsoQLR) — https://github.com/TBLabFJD/VIsoQLR
- **Minimap2** (Alternative aligner for long reads against a FASTA reference when GMAP is not used) — https://github.com/lh3/minimap2
- **TBLabFJD/VIsoQLR_benchmark** (Benchmark scripts including isoform_analysis_visoqlr_perc.R for comparing VIsoQLR results across read threshold values) — https://github.com/TBLabFJD/VIsoQLR_benchmark

## Examples

```
docker pull tblabfjd/visoqlr:latest; docker run -it -p 8888:8888 tblabfjd/visoqlr:latest
```

## Evaluation signals

- Reproduced isoform counts at thresholds 0.25–3% match the reported SIRV sweep: 65, 63, 59, 49, 41 isoforms out of 68 SIRV transcripts
- Cosine similarity between detected and reference SIRV isoform abundance profiles matches reported values 0.81, 0.81, 0.80, 0.78, 0.74 at thresholds 0.25–3%
- Detected isoform counts decrease monotonically as the read threshold increases
- VIsoQLR benchmark expectation: 72% of known SIRV isoforms detected with 99% exon coordinate accuracy, outperforming StringTie2 and FLAIR
- Mapped reads download from VIsoQLR yields both BAM and BAI index files

## Limitations

- Missing isoforms in automatic analysis are often below the default 2% threshold, so manual configuration of parameters is sometimes required
- Lowering the read threshold detects more isoforms at the cost of precision (more false positives from noise)
- Small isoform abundances (e.g., 4% and 1.7%) may be overrepresented by LRS and hardly distinguishable from noise in electropherograms
- No software version numbers reported for VIsoQLR, StringTie2, or FLAIR in the benchmark; no data accession identifiers given for the underlying LRS datasets
- For Windows users, the image requires Linux via WSL2 since it was developed for the Linux kernel

## Evidence

- [abstract] interactive analyzer, viewer and editor: "Here we describe VIsoQLR, an interactive analyzer, viewer and editor for the semi-automated identification and quantification of known and novel isoforms using long-read sequencing data."
- [methods] read threshold selects CECs: "The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads."
- [methods] docker pull command: "docker pull tblabfjd/visoqlr:latest"
- [methods] upload aligned reads: "navigate to your file system to upload your aligner reads in `GFF3`, `BED6`, or `BAM` file format."
- [discussion] default threshold tradeoff: "most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms"
- [readme] threshold sweep script in benchmark repo: [evidence span withheld — anchor status: not_found]
- [discussion] SIRV benchmark accuracy: "VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools."
