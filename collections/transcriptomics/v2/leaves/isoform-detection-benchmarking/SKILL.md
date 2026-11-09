---
name: isoform-detection-benchmarking
description: Use when evaluating or comparing isoform detection/quantification performance
  of long-read transcript tools against a known isoform annotation (e.g., SIRVs),
  or reproducing the VIsoQLR benchmark where VIsoQLR detects ~72% of isoforms with
  99% exon coordinate accuracy.
when_to_use_negative:
- The goal is genome-wide transcriptome assembly or differential expression rather
  than isoform-detection accuracy on a defined gene/spike-in locus — the wf-transcriptomes
  (bambu-based) reference-guided pipeline is designed for that scale instead.
- 'Only short-read data is available: VIsoQLR requires long-read (BAM/GFF3/BED6) alignments
  from technologies such as PacBio or Oxford Nanopore.'
- The target isoforms are expected below the default 2% CEC read threshold without
  the willingness to manually lower the threshold, since most missing isoforms in
  the article's analysis were below this value.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: VIsoQLR
  role: Interactive analyzer/viewer that identifies and quantifies isoforms from long-read
    BAM/GFF3/BED6 alignments without prior splice-site annotation; benchmarked against
    FLAIR and StringTie2
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: FLAIR
  role: Isoform detection tool run on the same alignment as a comparison method in
    the benchmark
  repo: https://github.com/TBLabFJD/VIsoQLR_benchmark
- name: StringTie2
  role: Transcript assembly tool run on the same alignment as a comparison method
    in the benchmark
  repo: https://github.com/TBLabFJD/VIsoQLR_benchmark
- name: Minimap2
  role: Splice-aware sequence mapper producing the shared BAM input when GMAP is not
    used ('-ax splice:hq -uf --secondary=no')
  repo: https://github.com/lh3/minimap2
- name: VIsoQLR_benchmark scripts
  role: Repository scripts (isoform_analysis.sh, isoform_comparison.R, isoform_analysis_visoqlr_perc.R)
    for data preprocessing, isoform calling, and cross-tool isoform comparison
  repo: https://github.com/TBLabFJD/VIsoQLR_benchmark
- name: Docker (tblabfjd/visoqlr)
  role: Containerized execution environment for VIsoQLR with port-mapped Jupyter-style
    access
  repo: https://hub.docker.com/r/tblabfjd/visoqlr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:c2f15e654e749cbc2b0290979cddef055e1d7ea96ca5a30b77bfe7d23c8c3c69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - Docker (tblabfjd/visoqlr)
  - FLAIR
  - Minimap2
  - StringTie2
  - VIsoQLR
  - VIsoQLR_benchmark scripts
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# isoform-detection-benchmarking

## Summary

Benchmark long-read isoform detection tools (VIsoQLR, FLAIR, StringTie2) on a ground-truth dataset such as the SIRV spike-in transcripts, comparing how many of the 68 known isoforms each tool recovers from the same alignment (GMAP or Minimap2 with '-ax splice:hq -uf --secondary=no').

## When to use

Use when evaluating or comparing isoform detection/quantification performance of long-read transcript tools against a known isoform annotation (e.g., SIRVs), or reproducing the VIsoQLR benchmark where VIsoQLR detects ~72% of isoforms with 99% exon coordinate accuracy.

## When NOT to use

- The goal is genome-wide transcriptome assembly or differential expression rather than isoform-detection accuracy on a defined gene/spike-in locus — the wf-transcriptomes (bambu-based) reference-guided pipeline is designed for that scale instead.
- Only short-read data is available: VIsoQLR requires long-read (BAM/GFF3/BED6) alignments from technologies such as PacBio or Oxford Nanopore.
- The target isoforms are expected below the default 2% CEC read threshold without the willingness to manually lower the threshold, since most missing isoforms in the article's analysis were below this value.

## Inputs

- PacBio long-read sequencing data (e.g., Sequel II UHR Iso-Seq SIRV reads from PacificBiosciences/DevNet)
- Reference genome sequence FASTA
- Known isoform annotation for the spike-in set (68 SIRV isoforms)
- BAM alignment of reads to the reference (GMAP- or minimap2-derived, with BAI index)

## Outputs

- Per-tool isoform detection counts versus known annotation (e.g., VIsoQLR 48/68 = 71%, FLAIR 35 = 51%, StringTie2 15 = 22%) tabulated as benchmark_detection_counts.csv
- Exon-coordinate comparison of detected isoforms (VIsoQLR ~99% exon coordinate accuracy on SIRVs)
- Optional sensitivity curve for VIsoQLR across read threshold (%) values

## How to apply

Obtain the ground-truth long-read dataset — in the article, PacBio SIRV reads from the Sequel II UHR Iso-Seq release (PacificBiosciences/DevNet) — plus the reference FASTA and the known SIRV isoform annotation. Align reads with a splice-aware mapper (GMAP via the VIsoQLR Docker panels, or Minimap2 using '-ax splice:hq -uf --secondary=no') to produce a BAM. Feed the same BAM independently to each tool: VIsoQLR via 'docker run -it -p 8888:8888 tblabfjd/visoqlr:latest' with its default parameters including the default 2% CEC read threshold; FLAIR and StringTie2 with default parameters (the TBLabFJD/VIsoQLR_benchmark 'isoform_analysis.sh' script performs preprocessing, GMAP index generation, and FLAIR/StringTie2 runs, while 'isoform_comparison.R' compares isoform sets). Intersect each tool's detected isoform exon-coordinate set against the known annotation and count correct detections; in the reproduced minimap2-based run VIsoQLR detected 48/68 (71%), FLAIR 35/68 (51%), StringTie2 15/68 (22%). Optionally repeat VIsoQLR across different read threshold values using 'isoform_analysis_visoqlr_perc.R' to assess sensitivity/specificity trade-offs of the CEC threshold.

## Related tools

- **VIsoQLR** (Interactive analyzer/viewer that identifies and quantifies isoforms from long-read BAM/GFF3/BED6 alignments without prior splice-site annotation; benchmarked against FLAIR and StringTie2) — https://github.com/TBLabFJD/VIsoQLR
- **FLAIR** (Isoform detection tool run on the same alignment as a comparison method in the benchmark) — https://github.com/TBLabFJD/VIsoQLR_benchmark
- **StringTie2** (Transcript assembly tool run on the same alignment as a comparison method in the benchmark) — https://github.com/TBLabFJD/VIsoQLR_benchmark
- **Minimap2** (Splice-aware sequence mapper producing the shared BAM input when GMAP is not used ('-ax splice:hq -uf --secondary=no')) — https://github.com/lh3/minimap2
- **VIsoQLR_benchmark scripts** (Repository scripts (isoform_analysis.sh, isoform_comparison.R, isoform_analysis_visoqlr_perc.R) for data preprocessing, isoform calling, and cross-tool isoform comparison) — https://github.com/TBLabFJD/VIsoQLR_benchmark
- **Docker (tblabfjd/visoqlr)** (Containerized execution environment for VIsoQLR with port-mapped Jupyter-style access) — https://hub.docker.com/r/tblabfjd/visoqlr

## Examples

```
minimap2 -ax splice:hq -uf --secondary=no reference.fa sirv_reads.fastq > aln.sam
```

## Evaluation signals

- VIsoQLR detects approximately 72% (48/68) of known SIRV isoforms with ~99% exon coordinate accuracy, outperforming FLAIR (35/68 ≈ 51%) and StringTie2 (15/68 ≈ 22%).
- Counts are reproduced with either GMAP or Minimap2 ('-ax splice:hq -uf --secondary=no') alignments — results should be similar across mappers.
- StringTie2-specific artifacts: it extends the boundaries of some isoforms relative to the BAM files and fails to call isoforms lacking certain exons (e.g., missed isoforms with missed exons 5 and 6).
- Missing VIsoQLR detections should mostly correspond to isoforms with read abundance below the default 2% CEC threshold; lowering the threshold should recover more isoforms at the cost of extra detections.
- The benchmark uses default parameters for FLAIR and StringTie2, per the isoform_analysis.sh script.

## Limitations

- Most isoforms missed by VIsoQLR's automatic analysis were below the default 2% read threshold; detecting them requires manual parameter tuning, which trades sensitivity for specificity.
- No software version numbers are reported for VIsoQLR, StringTie2, or FLAIR, limiting exact reproducibility.
- StringTie2's isoform boundary extension is an artifact not present in the BAM files, which can inflate apparent novel boundaries.
- The benchmark targets selected/individual genes and spike-in standards (SIRVs, PAX6 minigene), not a genome-wide transcriptome workflow.
- Minimap2 splice alignment has a documented limitation advised in its GitHub repository.

## Evidence

- [discussion] VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools.: "VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools."
- [methods] docker run command with port mapping: "docker run -it -p 8888:8888 tblabfjd/visoqlr:latest"
- [methods] BAM/GFF3/BED6 input formats: "navigate to your file system to upload your aligner reads in `GFF3`, `BED6`, or `BAM` file format."
- [discussion] Default 2% CEC read threshold trade-off: "most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms"
- [readme] Benchmark scripts README: [evidence span withheld — anchor status: external_doc]
- [discussion] StringTie2 boundary artifact: "This extension (also seen in the SIRVs analysis) seems to be an artifact as it is not present in BAM files used by StringTie2 and VIsoQLR"
