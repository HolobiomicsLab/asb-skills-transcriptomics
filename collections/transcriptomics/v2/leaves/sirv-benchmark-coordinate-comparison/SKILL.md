---
name: sirv-benchmark-coordinate-comparison
description: Use when evaluating the accuracy of a long-read isoform caller with known-truth
  spike-in transcripts, e.g. quantifying exon-coordinate accuracy and recall of known
  isoforms on SIRV data.
when_to_use_negative:
- The target transcripts have no ground-truth annotation — accuracy against known
  isoforms cannot be computed without references like the SIRV set
- The goal is de-novo transcript discovery in a poorly annotated genome rather than
  tool benchmarking against known truth
- Input is short-read sequencing data — this benchmark workflow is designed for long-read
  (PacBio/ONT) alignments
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: VIsoQLR
  role: Isoform detection and quantification being benchmarked; defines consensus
    splice sites and quantifies isoforms
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: StringTie2
  role: Comparison transcriptome-based tool run with default parameters in the benchmark
- name: FLAIR
  role: Comparison long-read isoform-detection tool run with default parameters in
    the benchmark
- name: GMAP
  role: Aligner used to build the reference index and align raw reads for isoform
    analysis
- name: VIsoQLR_benchmark
  role: Benchmark scripts for data preprocessing, isoform calling, and isoform comparison
  repo: https://github.com/TBLabFJD/VIsoQLR_benchmark
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:7184c1776ed1b9f33aa3e23570b859dde0f2c546df65c626756bfe08d6e05b90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - FLAIR
  - GMAP
  - StringTie2
  - VIsoQLR
  - VIsoQLR_benchmark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# SIRV benchmark coordinate comparison

## Summary

Benchmark isoform-detection accuracy on the SIRV spike-in control set by running VIsoQLR (vs. StringTie2 and FLAIR) on public PacBio Iso-Seq SIRV data and comparing detected exon coordinates and isoforms to the known SIRV transcript annotation.

## When to use

Use when evaluating the accuracy of a long-read isoform caller with known-truth spike-in transcripts, e.g. quantifying exon-coordinate accuracy and recall of known isoforms on SIRV data.

## When NOT to use

- The target transcripts have no ground-truth annotation — accuracy against known isoforms cannot be computed without references like the SIRV set
- The goal is de-novo transcript discovery in a poorly annotated genome rather than tool benchmarking against known truth
- Input is short-read sequencing data — this benchmark workflow is designed for long-read (PacBio/ONT) alignments

## Inputs

- Public PacBio Iso-Seq SIRV long-read dataset (Sequel II UHR release)
- SIRV reference transcript annotation (known isoforms and exon coordinates)
- BAM alignments of SIRV reads (GFF3, BED6, or BAM accepted by VIsoQLR)
- Benchmark scripts from TBLabFJD/VIsoQLR_benchmark

## Outputs

- Per-tool isoform calls for VIsoQLR, StringTie2, and FLAIR
- isoform comparison table/statistics (detected vs. known SIRV isoforms)
- Exon-coordinate accuracy metrics (e.g. 99% accuracy, 72% known-isoform recall)
- Read-threshold sensitivity analysis of VIsoQLR isoform detection

## How to apply

Acquire the public PacBio Sequel II UHR Iso-Seq data containing SIRV transcripts, preprocess it (GMAP index generation plus FLAIR and StringTie2 runs with default parameters, as scripted in the VIsoQLR_benchmark repository), and run VIsoQLR on the same alignments. VIsoQLR defines candidate consensus exon coordinates (CECs) by selecting splice-site coordinates above the Read threshold (default 2%), merging below-threshold CECs within a Padding window (default 5 bases), and collapsing candidate CECs closer than the Merge close splice sites distance (default 3 bases). Then compare the detected isoform set and exon coordinates against the SIRV reference annotation using the isoform_comparison.R script, recording both isoform recall (fraction of known isoforms detected) and exon-coordinate accuracy. Sweep the Read threshold (isoform_analysis_visoqlr_perc.R) to assess the sensitivity/specificity trade-off of missed low-abundance isoforms.

## Related tools

- **VIsoQLR** (Isoform detection and quantification being benchmarked; defines consensus splice sites and quantifies isoforms) — https://github.com/TBLabFJD/VIsoQLR
- **StringTie2** (Comparison transcriptome-based tool run with default parameters in the benchmark)
- **FLAIR** (Comparison long-read isoform-detection tool run with default parameters in the benchmark)
- **GMAP** (Aligner used to build the reference index and align raw reads for isoform analysis)
- **VIsoQLR_benchmark** (Benchmark scripts for data preprocessing, isoform calling, and isoform comparison) — https://github.com/TBLabFJD/VIsoQLR_benchmark

## Examples

```
bash isoform_analysis.sh && Rscript isoform_comparison.R && Rscript isoform_analysis_visoqlr_perc.R
```

## Evaluation signals

- VIsoQLR should detect ~72% of known SIRV isoforms with ~99% exon-coordinate accuracy, outperforming StringTie2 and FLAIR
- Most isoforms missed at the default 2% Read threshold should be below 2% abundance; lowering the threshold should recover more isoforms
- StringTie2-specific artifacts (extended isoform boundaries not present in the BAM files, missed-exon isoform failures) should be absent from VIsoQLR calls
- Detected CEC lists should validate against VIsoQLR's known SIRV CEC set (coordinate + frequency consistency)

## Limitations

- Missing isoforms in automatic VIsoQLR analysis are often those below the default 2% threshold, requiring manual parameter tuning per gene
- Small isoform abundances (e.g. 4% and 1.7%) may be overrepresented by LRS and are hard to distinguish from noise in reference electropherograms
- StringTie2 comparison results are affected by its own boundary-extension artifact, which may complicate cross-tool coordinate comparison

## Evidence

- [discussion] VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools.: "VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools."
- [methods] Read threshold selection of candidate CECs: "The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads."
- [readme] Benchmark data source on PacBio SIRV data: [evidence span withheld — anchor status: external_doc]
- [readme] Benchmark scope of VIsoQLR_benchmark repository: [evidence span withheld — anchor status: external_doc]
- [abstract] Benchmark conclusion from abstract: "A benchmark against two other popular transcriptome-based tools shows VIsoQLR accurate performance on both detection and quantification of isoforms."
