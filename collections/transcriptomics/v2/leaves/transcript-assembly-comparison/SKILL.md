---
name: transcript-assembly-comparison
description: Use when you must objectively compare isoform-detection performance of
  multiple transcript assembly tools on the same long-read dataset with a known ground-truth
  isoform annotation.
when_to_use_negative:
- When there is no reference annotation or ground-truth isoform set to compare against
  — sensitivity and exon-coordinate accuracy cannot be computed
- When analyzing a whole transcriptome de novo rather than a curated spike-in/benchmark
  locus set (SIRVs); VIsoQLR is designed for close inspection of one gene at a time
  in low-throughput splicing assays
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: VIsoQLR
  role: Interactive isoform detection and quantification from the shared BAM, with
    editable splice sites and an adjustable CEC read threshold (default 2%)
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: FLAIR
  role: Isoform detection without prior splice-site annotation, run on the same minimap2
    BAM with default parameters
- name: StringTie2
  role: Transcript isoform assembly on the same BAM; extends isoform boundaries in
    some cases (an artifact not present in the BAM)
- name: Minimap2
  role: Shared sequence mapper producing a single BAM ('-ax splice:hq -uf --secondary=no')
    consumed by all three tools for a fair comparison
  repo: https://github.com/lh3/minimap2
- name: TBLabFJD/VIsoQLR_benchmark
  role: Repository with the preprocessing, isoform-calling and comparison scripts
    (isoform_analysis.sh, isoform_comparison.R, isoform_analysis_visoqlr_perc.R)
  repo: TBLabFJD/VIsoQLR_benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:233633a9154bbe6f6f7d687bdf2ab0b07d964f76d180a4fcff94f0328e98ba3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - FLAIR
  - Minimap2
  - StringTie2
  - TBLabFJD/VIsoQLR_benchmark
  - VIsoQLR
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# transcript-assembly-comparison

## Summary

Benchmark long-read isoform detection tools (VIsoQLR, FLAIR, StringTie2) by aligning the same long reads with a shared mapper (GMAP or Minimap2 with '-ax splice:hq -uf --secondary=no'), running each tool on the identical BAM, and comparing detected isoform sets against a known annotation (e.g., 68 SIRV isoforms) to quantify sensitivity and exon-coordinate accuracy.

## When to use

Use when you must objectively compare isoform-detection performance of multiple transcript assembly tools on the same long-read dataset with a known ground-truth isoform annotation.

## When NOT to use

- When there is no reference annotation or ground-truth isoform set to compare against — sensitivity and exon-coordinate accuracy cannot be computed
- When analyzing a whole transcriptome de novo rather than a curated spike-in/benchmark locus set (SIRVs); VIsoQLR is designed for close inspection of one gene at a time in low-throughput splicing assays

## Inputs

- PacBio SIRV long reads (FASTQ) from the Sequel II UHR Iso-Seq public dataset
- Reference genome/transcript sequence (FASTA)
- Known isoform annotation (SIRV GTF) used as ground truth
- BAM alignment (plus BAI index) shared by all compared tools

## Outputs

- Per-tool detected isoform sets (GTF/transcript structures)
- Detection counts per tool vs. the 68 known SIRV isoforms (e.g., VIsoQLR 48/68 = 71%, FLAIR 35 = 51%, StringTie2 15 = 22%)
- Exon-coordinate accuracy per tool (VIsoQLR 99%)
- benchmark_detection_counts.csv tabulating counts per tool

## How to apply

First align the long reads once with a single mapper so all tools consume an identical BAM: either GMAP (index built from the reference FASTA) or Minimap2 with the high-quality splice-aware options '-ax splice:hq -uf --secondary=no'. Run VIsoQLR via its Docker image ('tblabfjd/visoqlr:latest', port 8888) uploading the BAM so it identifies and quantifies isoforms with default parameters, including the default 2% CEC read threshold; run FLAIR and StringTie2 on the same BAM with default parameters. Compare each tool's detected isoform set against the known SIRV isoform annotation and count correct detections (VIsoQLR 48/68 = 71%, FLAIR 35 = 51%, StringTie2 15 = 22%). Accept the benchmark if reproduced counts match the reported figures — VIsoQLR detecting ~72% of known isoforms with ~99% exon coordinate accuracy, outperforming FLAIR and StringTie2.

## Related tools

- **VIsoQLR** (Interactive isoform detection and quantification from the shared BAM, with editable splice sites and an adjustable CEC read threshold (default 2%)) — https://github.com/TBLabFJD/VIsoQLR
- **FLAIR** (Isoform detection without prior splice-site annotation, run on the same minimap2 BAM with default parameters)
- **StringTie2** (Transcript isoform assembly on the same BAM; extends isoform boundaries in some cases (an artifact not present in the BAM))
- **Minimap2** (Shared sequence mapper producing a single BAM ('-ax splice:hq -uf --secondary=no') consumed by all three tools for a fair comparison) — https://github.com/lh3/minimap2
- **TBLabFJD/VIsoQLR_benchmark** (Repository with the preprocessing, isoform-calling and comparison scripts (isoform_analysis.sh, isoform_comparison.R, isoform_analysis_visoqlr_perc.R)) — TBLabFJD/VIsoQLR_benchmark

## Examples

```
minimap2 -ax splice:hq -uf --secondary=no reference.fa sirv_reads.fq > aln.sam && docker run -it -p 8888:8888 tblabfjd/visoqlr:latest
```

## Evaluation signals

- Reproduced detection counts match reported figures: VIsoQLR 48/68 (71%), FLAIR 35 (51%), StringTie2 15 (22%) with minimap2 mapping
- VIsoQLR exon coordinate accuracy reaches ~99% against the SIRV annotation
- All tools were run on the same BAM (single mapper: GMAP or minimap2), not tool-specific alignments
- VIsoQLR benchmark used default parameters, including the default 2% CEC read threshold
- GMAP-based and minimap2-based runs yield similar results (VIsoQLR 48/68 in both)

## Limitations

- VIsoQLR's automatic analysis misses isoforms whose reads fall below the default 2% CEC read threshold; manual parameter tuning may be required
- StringTie2 extends isoform boundaries and fails to call isoforms lacking certain exons; the extension is an artifact not present in the BAM files
- No software version numbers are reported for VIsoQLR, StringTie2, or FLAIR in the benchmark
- Lowering the read threshold detects more isoforms at the cost of precision — a trade-off when tuning

## Evidence

- [discussion] VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools.: "VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools."
- [discussion] most missing isoforms were below 2% (default value): "most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms"
- [discussion] it extends the boundaries of some isoforms, and it does not call isoforms without certain exons: "it extends the boundaries of some isoforms, and it does not call isoforms without certain exons"
- [methods] docker run -it -p 8888:8888 tblabfjd/visoqlr:latest: "docker run -it -p 8888:8888 tblabfjd/visoqlr:latest"
- [readme] This repostory contains the scripts used for data preprocessing, isoform calling and isoform comparison of VIsoQLR, FLAIR and StringTie2.: [evidence span withheld — anchor status: external_doc]
