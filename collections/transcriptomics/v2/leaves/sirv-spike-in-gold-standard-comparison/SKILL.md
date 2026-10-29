---
name: sirv-spike-in-gold-standard-comparison
description: Use when evaluating the accuracy of an isoform detection/quantification
  method using SIRV spike-in transcripts with known isoform counts and gold-standard
  abundances as ground truth.
when_to_use_negative:
- When no spike-in transcripts with known isoform sequences and abundances are present
  in the sample, so no ground truth exists for detection counts or cosine similarity
- When the goal is de novo transcriptome annotation of a whole genome rather than
  benchmarking isoform recovery against a defined isoform set
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: VIsoQLR
  role: Isoform detection and quantification tool under test, run with default parameters
    via Docker on GMAP-aligned BAM input
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: GMAP
  role: Aligner producing the aligned reads (GFF3/BAM) used for isoform calling in
    the benchmark
- name: FLAIR
  role: Comparison tool benchmarked against VIsoQLR on SIRV spike-in detection
- name: StringTie2
  role: Comparison tool benchmarked against VIsoQLR on SIRV spike-in detection
- name: TBLabFJD/VIsoQLR_benchmark
  role: Scripts for data preprocessing, isoform calling and isoform comparison of
    VIsoQLR, FLAIR and StringTie2
  repo: https://github.com/TBLabFJD/VIsoQLR_benchmark
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:bd4586d5031a4f15f03dd14b41d3070dd0619d8faf1fffdb7af3ff4dead09dd3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - FLAIR
  - GMAP
  - StringTie2
  - TBLabFJD/VIsoQLR_benchmark
  - VIsoQLR
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# sirv-spike-in-gold-standard-comparison

## Summary

Benchmarks long-read isoform detection and quantification against Lexogen SIRV spike-in transcripts of known isoforms and abundances, as done for VIsoQLR vs FLAIR and StringTie2 on PacBio UHR Iso-Seq data.

## When to use

Use when evaluating the accuracy of an isoform detection/quantification method using SIRV spike-in transcripts with known isoform counts and gold-standard abundances as ground truth.

## When NOT to use

- When no spike-in transcripts with known isoform sequences and abundances are present in the sample, so no ground truth exists for detection counts or cosine similarity
- When the goal is de novo transcriptome annotation of a whole genome rather than benchmarking isoform recovery against a defined isoform set

## Inputs

- GMAP-aligned PacBio UHR Iso-Seq BAM file (e.g., cond_VIsoQLR_GMAP from TBLabFJD/VIsoQLR_benchmark)
- Lexogen SIRV reference transcripts (known isoforms with gold-standard abundances)
- Public PacBio Sequel-II UHR Iso-Seq RNA-seq data (PacificBiosciences/DevNet)

## Outputs

- Count of detected SIRV isoforms out of 68 known isoforms
- Exon coordinate accuracy percentage versus the SIRV annotation
- Cosine similarity between predicted and gold-standard isoform abundance profiles (overall and per SIRV set)
- Comparative isoform detection table across tools (VIsoQLR, FLAIR, StringTie2)

## How to apply

Align long reads to the reference (e.g., GMAP for PacBio Iso-Seq BAM input) and run the isoform caller under test with default parameters — for VIsoQLR this means a 2% read threshold with default padding and merge-close-splice-sites settings for consensus exon coordinate detection. Upload the Lexogen SIRV reference transcripts as known isoforms in the comparison panel, then export the isoform information panel and exonic starting/ending points to count how many of the 68 SIRV isoforms were detected. Compute exon coordinate accuracy against the SIRV annotation and cosine similarity between the predicted isoform abundance profile and the Lexogen gold standard abundances, both overall and per SIRV mixture (e.g., SIRV7). Compare detection counts and similarities across tools (VIsoQLR vs FLAIR vs StringTie2) using shared preprocessing scripts such as those in TBLabFJD/VIsoQLR_benchmark.

## Related tools

- **VIsoQLR** (Isoform detection and quantification tool under test, run with default parameters via Docker on GMAP-aligned BAM input) — https://github.com/TBLabFJD/VIsoQLR
- **GMAP** (Aligner producing the aligned reads (GFF3/BAM) used for isoform calling in the benchmark)
- **FLAIR** (Comparison tool benchmarked against VIsoQLR on SIRV spike-in detection)
- **StringTie2** (Comparison tool benchmarked against VIsoQLR on SIRV spike-in detection)
- **TBLabFJD/VIsoQLR_benchmark** (Scripts for data preprocessing, isoform calling and isoform comparison of VIsoQLR, FLAIR and StringTie2) — https://github.com/TBLabFJD/VIsoQLR_benchmark

## Examples

```
docker pull tblabfjd/visoqlr:latest && docker run -it -p 8888:8888 tblabfjd/visoqlr:latest
```

## Evaluation signals

- VIsoQLR detects 49 of the 68 SIRV isoforms (72%) with default 2% read threshold, versus FLAIR (37; 54%) and StringTie2 (12; 18%)
- Exon coordinate accuracy of detected SIRV isoforms reaches 99%
- Overall cosine similarity between VIsoQLR abundance profile and Lexogen gold standard is 0.78, with partial similarity highest for SIRV7 (0.97)
- Most missed isoforms fall below the default 2% read threshold, so lowering the threshold should increase detected isoform count
- Comparison scripts (isoform_comparison.R, isoform_analysis_visoqlr_perc.R) reproduce cross-tool and threshold-sweep results

## Limitations

- Isoforms with abundance below the default 2% read threshold are missed by VIsoQLR's automatic analysis; detection requires manual parameter tuning at the cost of detecting more spurious isoforms
- StringTie2 artifact: it extends isoform boundaries beyond what is present in the BAM files used by both StringTie2 and VIsoQLR
- Small isoform abundances (e.g., 4% and 1.7%) may be overrepresented by LRS and hard to distinguish from noise in electropherograms

## Evidence

- [discussion] VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools.: "VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools."
- [readme] SIRV transcripts were analyced on a public PacBio RNA-seq Data (https://github.com/PacificBiosciences/DevNet/wiki/Sequel-II-System-Data-Release:-Universal-Human-Reference-(UHR)-Iso-Seq).: [evidence span withheld — anchor status: external_doc]
- [methods] The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads.: "The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads."
- [discussion] most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms: "most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms"
- [discussion] This extension (also seen in the SIRVs analysis) seems to be an artifact as it is not present in BAM files used by StringTie2 and VIsoQLR: "This extension (also seen in the SIRVs analysis) seems to be an artifact as it is not present in BAM files used by StringTie2 and VIsoQLR"
