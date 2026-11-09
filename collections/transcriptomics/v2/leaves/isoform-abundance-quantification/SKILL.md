---
name: isoform-abundance-quantification
description: Use when a GMAP- or minimap2-aligned BAM from long-read (PacBio Iso-Seq
  or ONT) data needs per-isoform abundance quantification for a single gene/locus,
  and the read-threshold parameter must be tuned or its sensitivity characterized
  against a known isoform profile.
when_to_use_negative:
- The goal is whole-transcriptome annotation rather than close inspection and fine-tuning
  of one gene at a time — VIsoQLR is explicitly designed for single-locus, low-throughput
  splicing assays.
- Input is raw FASTQ that has not yet been aligned (GMAP/minimap2 alignment must precede
  isoform identification and quantification).
- Isoform quantification accuracy must be judged by exon-coordinate correctness alone
  (e.g. the SIRV benchmark's 99% exon-coordinate accuracy) rather than by abundance-profile
  similarity, which this skill measures.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: VIsoQLR
  role: Interactive analyzer that defines consensus splice sites/CECs and quantifies
    isoforms from aligned long reads; its 'Read threshold (%)' CEC parameter is swept
    to assess sensitivity
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: GMAP
  role: Aligner used upstream to map long cDNA reads to the reference, producing the
    BAM consumed by VIsoQLR
- name: VIsoQLR_benchmark
  role: Contains isoform_analysis_visoqlr_perc.R, the script for isoform comparison
    across different VIsoQLR read-threshold values
  repo: https://github.com/TBLabFJD/VIsoQLR_benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:637fdbda068576d56098e0e98e8a82b026525d828b724a098501da47be5b2979
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - GMAP
  - VIsoQLR
  - VIsoQLR_benchmark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# isoform-abundance-quantification

## Summary

Quantify isoform abundances from long-read sequencing alignments of a target gene using VIsoQLR's CEC-based isoform identification, testing how the 'Read threshold (%)' parameter trades off isoform sensitivity against quantification accuracy (cosine similarity to a reference abundance profile).

## When to use

Use when a GMAP- or minimap2-aligned BAM from long-read (PacBio Iso-Seq or ONT) data needs per-isoform abundance quantification for a single gene/locus, and the read-threshold parameter must be tuned or its sensitivity characterized against a known isoform profile.

## When NOT to use

- The goal is whole-transcriptome annotation rather than close inspection and fine-tuning of one gene at a time — VIsoQLR is explicitly designed for single-locus, low-throughput splicing assays.
- Input is raw FASTQ that has not yet been aligned (GMAP/minimap2 alignment must precede isoform identification and quantification).
- Isoform quantification accuracy must be judged by exon-coordinate correctness alone (e.g. the SIRV benchmark's 99% exon-coordinate accuracy) rather than by abundance-profile similarity, which this skill measures.

## Inputs

- GMAP-aligned BAM file (plus BAI index) of long-read cDNA reads for the target gene/locus (e.g. SIRV spike-in transcripts)
- Reference isoform abundance profile (e.g. SIRV transcript mix) for cosine-similarity comparison
- Docker (or Docker Desktop with WSL2 on Windows)

## Outputs

- Number of detected isoforms at each read-threshold setting (e.g. 65/63/59/49/41 of 68 SIRV transcripts at 0.25%–3%)
- Cosine similarity of detected vs. reference abundance profile per threshold (e.g. 0.81/0.81/0.80/0.78/0.74)
- sirv_threshold_sweep_results.csv summarizing counts and similarities across thresholds

## How to apply

Pull and launch the VIsoQLR Docker image (docker pull tblabfjd/visoqlr:latest; docker run -it -p 8888:8888) and open the web interface on port 8888. Upload the aligned reads in BAM (GFF3 or BED6 also accepted) via the 'Browse...' button so VIsoQLR runs its default consensus-exon-coordinate (CEC) identification and quantification pipeline. In the 'Exon coordinates: Automatic detection' panel, set the 'Read threshold (%)' — the proportion of reads supporting candidate exon coordinates — sweeping it across 0.25, 0.5, 1, 2, and 3 percent, recording the number of detected isoforms at each value. At each threshold, quantify isoform abundances and compute the cosine similarity between the detected isoform abundance profile and the reference (SIRV) profile. Lowering the threshold recovers more low-abundance isoforms (most missed isoforms lie below the 2% default) but increases sensitivity to noise, so report both count and similarity to judge the setting. Assemble counts and similarities into a results table (e.g. sirv_threshold_sweep_results.csv).

## Related tools

- **VIsoQLR** (Interactive analyzer that defines consensus splice sites/CECs and quantifies isoforms from aligned long reads; its 'Read threshold (%)' CEC parameter is swept to assess sensitivity) — https://github.com/TBLabFJD/VIsoQLR
- **GMAP** (Aligner used upstream to map long cDNA reads to the reference, producing the BAM consumed by VIsoQLR)
- **VIsoQLR_benchmark** (Contains isoform_analysis_visoqlr_perc.R, the script for isoform comparison across different VIsoQLR read-threshold values) — https://github.com/TBLabFJD/VIsoQLR_benchmark

## Examples

```
docker pull tblabfjd/visoqlr:latest && docker run -it -p 8888:8888 tblabfjd/visoqlr:latest
```

## Evaluation signals

- Detected isoform counts at thresholds 0.25%, 0.5%, 1%, 2%, and 3% match the reported 65, 63, 59, 49, and 41 (of 68 SIRV transcripts).
- Cosine similarities between detected and reference abundance profiles match the reported 0.81, 0.81, 0.80, 0.78, and 0.74 across the same thresholds.
- Isoform count decreases monotonically as the read threshold increases (0.25% → 3%).
- The sweep results file (sirv_threshold_sweep_results.csv) contains five thresholds × (count, cosine similarity) rows with no missing values.
- At the default 2% threshold, missing reference isoforms should predominantly be those with abundances below 2%, consistent with the article's explanation.

## Limitations

- Lowering the read threshold detects more isoforms but at the cost of quantification accuracy (cosine similarity drops from 0.81 at 0.25% to 0.74 at 3%-threshold regime) and risks calling noise as isoforms.
- Small isoform abundances (e.g. 4% and 1.7%) may be overrepresented by long-read sequencing and are hard to distinguish from noise in electropherograms.
- No software version numbers are reported for VIsoQLR, complicating exact reproducibility; the automatic analysis may still require manual configuration of certain parameters per gene.

## Evidence

- [methods] Read threshold definition: "The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads."
- [discussion] Default threshold trade-off: "most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms"
- [methods] Docker invocation: "docker run -it -p 8888:8888 tblabfjd/visoqlr:latest"
- [readme] Benchmark script for threshold sweep: [evidence span withheld — anchor status: not_found]
- [intro] Interactive single-gene scope: "there is no interactive tool that allows a close inspection and fine-tuning analysis of one gene at a time on data from long-reads RNA sequencing."
