---
name: consensus-exon-coordinate-threshold-tuning
description: Use when a single-locus long-read RNA-seq analysis (BAM/GFF3/BED6 input)
  misses known isoforms under VIsoQLR's default 2% CEC threshold, or when a documented
  sensitivity/quantification trade-off curve for the threshold is needed before manual
  curation.
when_to_use_negative:
- Goal is whole-transcriptome de novo isoform discovery across many genes — VIsoQLR
  is designed for close inspection and fine-tuning of one gene at a time in low-throughput
  splicing assays
- Input is raw FASTQ with no reference-guided alignment step performed — build the
  GMAP index and align first, or the CEC proportions among mapped reads cannot be
  computed
- All target isoforms have abundances well above the default 2% and quantification
  already matches the reference — threshold tuning adds no information
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VIsoQLR
  role: Interactive analyzer whose 'Read threshold (%)' CEC parameter is swept; runs
    isoform identification and quantification per gene
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: GMAP
  role: Aligner used to produce the BAM input; VIsoQLR builds a GMAP reference index
    and can align FASTQ reads with --format=2 to generate exon-coordinate GFF3
- name: VIsoQLR_benchmark
  role: Provides isoform_analysis_visoqlr_perc.R, the script comparing VIsoQLR results
    across different read threshold values
  repo: https://github.com/TBLabFJD/VIsoQLR_benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:25f14d961cf410de0720f9894eac950a9589f2276c1abbfabfd72289b53ebef9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - GMAP
  - VIsoQLR
  - VIsoQLR_benchmark
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# consensus-exon-coordinate-threshold-tuning

## Summary

Sweeps the VIsoQLR 'Read threshold (%)' parameter used to select candidate consensus exon coordinates (CECs) to balance isoform detection sensitivity against quantification accuracy for a single gene. Lower thresholds rescue low-abundance isoforms but admit noise; this skill quantifies that trade-off explicitly.

## When to use

Use when a single-locus long-read RNA-seq analysis (BAM/GFF3/BED6 input) misses known isoforms under VIsoQLR's default 2% CEC threshold, or when a documented sensitivity/quantification trade-off curve for the threshold is needed before manual curation.

## When NOT to use

- Goal is whole-transcriptome de novo isoform discovery across many genes — VIsoQLR is designed for close inspection and fine-tuning of one gene at a time in low-throughput splicing assays
- Input is raw FASTQ with no reference-guided alignment step performed — build the GMAP index and align first, or the CEC proportions among mapped reads cannot be computed
- All target isoforms have abundances well above the default 2% and quantification already matches the reference — threshold tuning adds no information

## Inputs

- Long-read sequencing reads aligned to a reference genome (BAM with BAI index; GFF3 or BED6 also accepted)
- Reference isoform abundance profile for cosine-similarity scoring (e.g. 68 SIRV transcripts)
- VIsoQLR Docker image (tblabfjd/visoqlr:latest) with web interface on port 8888

## Outputs

- Per-threshold isoform counts (e.g. 65, 63, 59, 49, 41 detected SIRV isoforms at 0.25–3%)
- Per-threshold cosine similarity between detected and reference abundance profiles (e.g. 0.81, 0.81, 0.80, 0.78, 0.74)
- Selected 'Read threshold (%)' value with justified detection/quantification trade-off, assembled into a results table (e.g. sirv_threshold_sweep_results.csv)

## How to apply

Launch the VIsoQLR Docker container (docker pull tblabfjd/visoqlr:latest; docker run -it -p 8888:8888) and upload a GMAP-aligned BAM file so the default isoform identification and quantification pipeline runs. In the 'Exon coordinates: Automatic detection' panel, set 'Read threshold (%)' — the parameter that selects candidate CECs based on the proportion of exon coordinates among all reads, default 2% — to a low value such as 0.25, then repeat at 0.5, 1, 2, and 3, recording the number of detected isoforms at each setting. At each threshold, quantify isoform abundances and compute the cosine similarity between the detected isoform abundance profile and a reference profile (e.g. SIRV transcripts). Choose the threshold where the count increase flattens while cosine similarity remains high; on 68 SIRV transcripts, sweeping 0.25–3% yields 65, 63, 59, 49, and 41 isoforms with cosine similarities 0.81, 0.81, 0.80, 0.78, and 0.74. Related companion parameters worth adjusting in the same panel are 'Padding (# of bases)' and 'Merge close splice sites (# of bases)'. The benchmark repository's isoform_analysis_visoqlr_perc.R script automates the cross-threshold comparison.

## Related tools

- **VIsoQLR** (Interactive analyzer whose 'Read threshold (%)' CEC parameter is swept; runs isoform identification and quantification per gene) — https://github.com/TBLabFJD/VIsoQLR
- **GMAP** (Aligner used to produce the BAM input; VIsoQLR builds a GMAP reference index and can align FASTQ reads with --format=2 to generate exon-coordinate GFF3)
- **VIsoQLR_benchmark** (Provides isoform_analysis_visoqlr_perc.R, the script comparing VIsoQLR results across different read threshold values) — https://github.com/TBLabFJD/VIsoQLR_benchmark

## Examples

```
docker run -it -p 8888:8888 tblabfjd/visoqlr:latest
```

## Evaluation signals

- Isoform counts increase monotonically as the read threshold is lowered (SIRV expectation: 41 at 3% up to 65 of 68 at 0.25%)
- Cosine similarity against the reference abundance profile stays high at low thresholds and degrades with over-detection (SIRV expectation: 0.81 at 0.25% falling to 0.74 at 3%)
- Counts and similarities match the reported reference values (65, 63, 59, 49, 41; 0.81, 0.81, 0.80, 0.78, 0.74) at thresholds 0.25, 0.5, 1, 2, 3 percent
- Detected isoforms' exon coordinates agree with the reference (VIsoQLR reports 99% exon-coordinate accuracy on SIRVs)
- All five threshold replicates produce a populated results table (sirv_threshold_sweep_results.csv) with paired count and similarity columns

## Limitations

- Lowering the threshold detects more isoforms at the cost of including noise; most isoforms missing from the default automatic analysis were below the 2% default value
- Small isoform abundances (e.g. 4% and 1.7% in the PAX6 case study) may be overrepresented by long-read sequencing and are hard to distinguish from noise in electropherograms
- Isoform collection for individual genes often still requires close inspection and manual configuration of other parameters after threshold tuning
- No software version numbers were reported for the benchmarked tools, so exact reproducibility across VIsoQLR releases is not guaranteed

## Evidence

- [methods] Read threshold selects candidate CECs by exon-coordinate proportion: "The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads."
- [discussion] Default 2% threshold trade-off between missing and over-detecting isoforms: "most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms"
- [discussion] Manual configuration needed for per-gene isoform collection: "the description of the isoforms collection of individual genes seems to require a close inspection and sometimes a manual configuration of certain parameters"
- [readme] Benchmark script for cross-threshold comparison: [evidence span withheld — anchor status: not_found]
