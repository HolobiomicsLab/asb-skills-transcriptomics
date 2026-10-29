---
name: sirv-benchmark-reproducibility-analysis
description: Use when quantifying how a VIsoQLR parameter (e.g. read threshold 0.25%–3%)
  affects isoform detection counts and quantification accuracy on SIRV spike-in benchmarks,
  or when validating a published VIsoQLR run.
when_to_use_negative:
- The goal is transcriptome-wide discovery or differential expression across many
  samples — VIsoQLR is designed for close inspection and fine-tuning of one gene (or
  a small SIRV set) at a time in low-throughput splicing assays; use a workflow like
  epi2me-labs/wf-transcriptomes instead.
- Input reads are unaligned FASTQ and no GMAP index is available — this skill assumes
  an already GMAP-aligned BAM; first perform alignment via the VIsoQLR GMAP panels
  or another aligner.
- The input is not a SIRV or other spike-in with a known reference isoform profile
  — cosine-similarity accuracy assessment requires a ground-truth abundance profile.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: VIsoQLR
  role: Interactive analyzer/viewer/editor used to identify and quantify isoforms
    while sweeping the 'Read threshold (%)' CEC parameter
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: GMAP
  role: Aligner used to produce the BAM input of SIRV reads (GFF3/BAM output with
    exon coordinates)
- name: TBLabFJD/VIsoQLR_benchmark
  role: Repository with isoform_analysis_visoqlr_perc.R, the script comparing VIsoQLR
    results across different read threshold values
  repo: https://github.com/TBLabFJD/VIsoQLR_benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:be7fd3e6c718007d6e2b35caaf9e9f4fc2a3634d29f6872f591f9f9126349dae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - GMAP
  - TBLabFJD/VIsoQLR_benchmark
  - VIsoQLR
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# sirv-benchmark-reproducibility-analysis

## Summary

Reproduces a parameter-sensitivity benchmark of VIsoQLR isoform detection by sweeping the 'Read threshold (%)' CEC parameter over SIRV long-read data and comparing detected isoform counts and quantification accuracy (cosine similarity) against published reference values.

## When to use

Use when quantifying how a VIsoQLR parameter (e.g. read threshold 0.25%–3%) affects isoform detection counts and quantification accuracy on SIRV spike-in benchmarks, or when validating a published VIsoQLR run.

## When NOT to use

- The goal is transcriptome-wide discovery or differential expression across many samples — VIsoQLR is designed for close inspection and fine-tuning of one gene (or a small SIRV set) at a time in low-throughput splicing assays; use a workflow like epi2me-labs/wf-transcriptomes instead.
- Input reads are unaligned FASTQ and no GMAP index is available — this skill assumes an already GMAP-aligned BAM; first perform alignment via the VIsoQLR GMAP panels or another aligner.
- The input is not a SIRV or other spike-in with a known reference isoform profile — cosine-similarity accuracy assessment requires a ground-truth abundance profile.

## Inputs

- GMAP-aligned SIRV BAM file (plus BAI index) from long-read (PacBio/ONT) sequencing of SIRV spike-in transcripts
- SIRV reference isoform abundance profile (68 known SIRV transcripts)
- VIsoQLR Docker image tblabfjd/visoqlr:latest

## Outputs

- Detected SIRV isoform counts at each read threshold (0.25%, 0.5%, 1%, 2%, 3%)
- Cosine similarity between detected and reference SIRV isoform abundance profiles at each threshold
- sirv_threshold_sweep_results.csv summarizing counts and similarities

## How to apply

Launch the tblabfjd/visoqlr Docker image (docker pull tblabfjd/visoqlr:latest; docker run -it -p 8888:8888) and open the web interface on port 8888. Upload the GMAP-aligned SIRV BAM file so VIsoQLR runs its default isoform identification and quantification pipeline. In the 'Exon coordinates: Automatic detection' panel, set 'Read threshold (%)' to 0.25 and record the number of detected isoforms, then repeat for 0.5, 1, 2, and 3 percent. At each threshold, quantify isoform abundances and compute the cosine similarity between the detected SIRV isoform abundance profile and the reference isoform profile. The rationale is that the default 2% threshold misses low-abundance isoforms (most missing isoforms were below 2%), so lowering it recovers more isoforms at some cost in quantification fidelity. Assemble the five counts and five cosine similarities into a CSV (e.g. sirv_threshold_sweep_results.csv) and compare against the expected reference values.

## Related tools

- **VIsoQLR** (Interactive analyzer/viewer/editor used to identify and quantify isoforms while sweeping the 'Read threshold (%)' CEC parameter) — https://github.com/TBLabFJD/VIsoQLR
- **GMAP** (Aligner used to produce the BAM input of SIRV reads (GFF3/BAM output with exon coordinates))
- **TBLabFJD/VIsoQLR_benchmark** (Repository with isoform_analysis_visoqlr_perc.R, the script comparing VIsoQLR results across different read threshold values) — https://github.com/TBLabFJD/VIsoQLR_benchmark

## Examples

```
docker pull tblabfjd/visoqlr:latest && docker run -it -p 8888:8888 tblabfjd/visoqlr:latest
```

## Evaluation signals

- Detected isoform counts at thresholds 0.25%–3% match the expected (65, 63, 59, 49, 41) of 68 SIRV transcripts
- Cosine similarities at thresholds 0.25%–3% match the expected (0.81, 0.81, 0.80, 0.78, 0.74)
- Counts decrease monotonically as the read threshold increases
- Cosine similarity stays ≥ 0.74 across the sweep and declines as the threshold rises
- Result CSV contains exactly five thresholds paired with one count and one cosine similarity each

## Limitations

- The default 2% read threshold misses low-abundance isoforms; lowering it detects more isoforms but may include noise-level candidates — manual configuration of parameters is sometimes required.
- Small isoform abundances (e.g. ~4% and 1.7%) may be overrepresented by long-read sequencing and hard to distinguish from noise in electropherograms.
- Accuracy metrics are specific to the SIRV spike-in (68 transcripts); results do not transfer directly to genes without a reference isoform profile.
- Minimap2 has alignment limitations for this data type noted in its GitHub; GMAP is the aligner used for the published SIRV analysis.

## Evidence

- [methods] Read threshold parameter sensitivity: "The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads."
- [discussion] Default threshold trade-off: "most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms"
- [methods] Docker invocation: "docker pull tblabfjd/visoqlr:latest"
- [readme] Benchmark script exists: [evidence span withheld — anchor status: not_found]
- [abstract] VIsoQLR core function: "Our tool takes sequences aligned to a reference, and for each gene, it defines consensus splice sites and quantifies isoforms."
