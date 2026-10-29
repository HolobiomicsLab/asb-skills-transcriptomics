---
name: coverage-limit-sensitivity-analysis
description: Use when comparing fusion-detection tools on simulated paired-end RNA-seq
  reads titrated across coverage levels, to rank methods by the lowest coverage at
  which each still detects a known fusion such as BCR-ABL1 or CIC-DUX4.
when_to_use_negative:
- Detecting fusion genes not involving the user-provided anchored target gene — Anchored-fusion
  only reports fusions that include the anchored gene
- Single-end RNA-seq, whole-genome, or whole-exome sequencing data — Anchored-fusion
  requires paired-end RNA-seq
- Discovery of novel/unspecified fusions — this skill requires pre-simulated, known
  fusion events with known breakpoints to compute coverage limits
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3522
- http://edamontology.org/topic_0085
tools:
- name: Anchored-fusion
  role: Anchored targeted fusion detector being benchmarked; anchors the gene of interest
    (BCR-ABL1, CIC-DUX4, DUX4-IGH) and recovers non-unique short-read matches for
    sensitive low-coverage detection
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: STAR-Fusion
  role: Competing bulk RNA-seq fusion detector (v1.12.0) run on the same simulated
    FASTQ inputs for coverage-limit comparison
  repo: https://github.com/STAR-Fusion
- name: FusionInspector
  role: Competing bulk RNA-seq fusion detector (v2.8.0) run on the same simulated
    FASTQ inputs for coverage-limit comparison
  repo: https://github.com/FusionInspector/FusionInspector
- name: FusionCatcher
  role: Competing bulk RNA-seq fusion detector (v1.30) whose DUX4-IGH detection coverage
    limit serves as the 4x reference point for Anchored-fusion sensitivity
  repo: https://github.com/ndaniel/fusioncatcher
- name: Arriba
  role: Competing bulk RNA-seq fusion detector (v2.4.0) run on the same simulated
    FASTQ inputs for coverage-limit comparison
  repo: https://github.com/suhrig/arriba
- name: BWA-MEM
  role: Alignment engine used by Anchored-fusion to map paired-end reads to the anchored
    gene reference and the whole genome
- name: BLAT
  role: Local alignment of candidate chimeric reads to pinpoint the precise fusion
    location before counting spanning/split read support
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:8846108440574eb67c173e882235a91c337bb7b0a7798eea5c390707c6e5d51b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  repo_url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  tools:
  - Anchored-fusion
  - Arriba
  - BLAT
  - BWA
  - BWA-MEM
  - FusionCatcher
  - FusionInspector
  - PyTorch
  - STAR-Fusion
  - bedtools
  - samtools
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3522
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# coverage-limit-sensitivity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that determines the minimum simulated read coverage at which each fusion-detection tool (Anchored-fusion vs. STAR-Fusion, FusionInspector, FusionCatcher, Arriba) first calls a target fusion event, quantifying relative detection sensitivity for events like BCR-ABL1 and CIC-DUX4.

## When to use

Use when comparing fusion-detection tools on simulated paired-end RNA-seq reads titrated across coverage levels, to rank methods by the lowest coverage at which each still detects a known fusion such as BCR-ABL1 or CIC-DUX4.

## When NOT to use

- Detecting fusion genes not involving the user-provided anchored target gene — Anchored-fusion only reports fusions that include the anchored gene
- Single-end RNA-seq, whole-genome, or whole-exome sequencing data — Anchored-fusion requires paired-end RNA-seq
- Discovery of novel/unspecified fusions — this skill requires pre-simulated, known fusion events with known breakpoints to compute coverage limits

## Inputs

- Simulated paired-end FASTQ files (e.g. test_sample_1.fastq.gz) covering target fusion transcripts at serially titrated coverage depths
- Target gene transcript FASTA (downloaded via NCBI Gene) for the anchored gene
- Genome FASTA (hg38 or hg19) and compatible gene annotation GTF
- Simulated reads combined with real RNA-seq background (e.g. GM12878 RNA-seq sample)

## Outputs

- Per-tool counts of spanning reads and split reads supporting each simulated fusion at each coverage level
- Minimum detection coverage per tool per fusion (e.g. DUX4-IGH limit for Anchored-fusion = 1/4 of FusionCatcher's)
- Compiled benchmark table (simulated_fusion_benchmark_results.csv) summarizing sensitivity rankings

## How to apply

Simulate paired-end FASTQ reads spanning fusion transcripts at serially decreasing coverage depths and run each detector on the same inputs: Anchored-Fusion (anchored search via BWA-MEM >=0.7.17, BLAT >=v.35, samtools >=1.16, bedtools >=v2.30.0) alongside STAR-Fusion v1.12.0, FusionInspector v2.8.0, FusionCatcher v1.30, and Arriba v2.4.0. For each tool and each simulated event (e.g. BCR-ABL1, CIC-DUX4, DUX4-IGH), tally supporting spanning reads and split reads at every coverage level and record the minimum coverage at which the fusion is still called. The tool with the lowest detection coverage limit is judged the most sensitive; in the source study Anchored-fusion achieved the best detection of BCR-ABL1 and CIC-DUX4 and detected DUX4-IGH at one-fourth of FusionCatcher's coverage limit. Compile per-tool, per-fusion coverage limits into a results table (e.g. simulated_fusion_benchmark_results.csv, corresponding to Figure 3A/3B) and validate against the expected ordering before drawing sensitivity conclusions.

## Related tools

- **Anchored-fusion** (Anchored targeted fusion detector being benchmarked; anchors the gene of interest (BCR-ABL1, CIC-DUX4, DUX4-IGH) and recovers non-unique short-read matches for sensitive low-coverage detection) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **STAR-Fusion** (Competing bulk RNA-seq fusion detector (v1.12.0) run on the same simulated FASTQ inputs for coverage-limit comparison) — https://github.com/STAR-Fusion
- **FusionInspector** (Competing bulk RNA-seq fusion detector (v2.8.0) run on the same simulated FASTQ inputs for coverage-limit comparison) — https://github.com/FusionInspector/FusionInspector
- **FusionCatcher** (Competing bulk RNA-seq fusion detector (v1.30) whose DUX4-IGH detection coverage limit serves as the 4x reference point for Anchored-fusion sensitivity) — https://github.com/ndaniel/fusioncatcher
- **Arriba** (Competing bulk RNA-seq fusion detector (v2.4.0) run on the same simulated FASTQ inputs for coverage-limit comparison) — https://github.com/suhrig/arriba
- **BWA-MEM** (Alignment engine used by Anchored-fusion to map paired-end reads to the anchored gene reference and the whole genome)
- **BLAT** (Local alignment of candidate chimeric reads to pinpoint the precise fusion location before counting spanning/split read support)

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=target_gene.fasta --fastq1=test_sample_1.fastq.gz --fastq2=test_sample_1.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- Anchored-fusion shows the lowest minimum detection coverage (highest sensitivity) for both BCR-ABL1 and CIC-DUX4 across all five tools
- Anchored-fusion's DUX4-IGH detection coverage limit equals exactly one-fourth of FusionCatcher's limit
- Per-tool spanning-read and split-read tallies are non-decreasing with increasing simulated coverage for each fusion event
- All tools were run on identical simulated paired-end FASTQ inputs and reference/annotation versions, ensuring a fair comparison
- Compiled results table matches the ordering and values reported in Figure 3A/3B of the source study

## Limitations

- Fusion genes that do not include the user-provided target gene will not be detected by Anchored-fusion
- Anchored-fusion can only detect fusion genes based on paired-end RNA-seq data and cannot be used for single-end RNA-seq or whole-genome/whole-exome sequencing data
- If the user dataset is too small, the default pre-trained HVLD model parameters from GitHub are recommended instead of retraining, which may deviate on the new data

## Evidence

- [results] Among all methods tested on simulated fusion transcripts, Anchored-fusion demonstrated the best detection capability for the BCR-ABL1 and CIC-DUX4 fusion genes: [evidence span withheld — anchor status: not_found]
