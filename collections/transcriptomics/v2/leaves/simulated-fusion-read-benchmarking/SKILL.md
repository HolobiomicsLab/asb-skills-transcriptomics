---
name: simulated-fusion-read-benchmarking
description: Use when comparing fusion gene detection sensitivity across tools on
  simulated paired-end RNA-seq reads, especially to establish minimum detection coverage
  for specific fusion genes like BCR-ABL1 or CIC-DUX4.
when_to_use_negative:
- The data are single-end RNA-seq, whole-genome, or whole-exome sequencing data —
  Anchored-fusion only supports paired-end RNA-seq
- The goal is de novo discovery of fusions that do not include a user-specified anchor/target
  gene — Anchored-fusion only reports fusions containing the anchored gene
- Detecting fusions without a known target gene of interest, since Anchored-fusion
  requires the user to supply the reference sequence of the target gene
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: Anchored-fusion
  role: Targeted fusion detector under benchmark; anchors a gene of interest and recovers
    non-unique short-read matches, with optional HVLD deep learning false-positive
    filtering
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: STAR-Fusion
  role: Competing bulk RNA-seq fusion detection method (v1.12.0) run on the same simulated
    fastq inputs
  repo: https://github.com/STAR-Fusion
- name: FusionInspector
  role: Competing bulk RNA-seq fusion detection method (v2.8.0) run on the same simulated
    fastq inputs
  repo: https://github.com/FusionInspector/FusionInspector
- name: FusionCatcher
  role: Competing bulk RNA-seq fusion detection method (v1.30); its coverage limit
    for DUX4-IGH is the reference against which Anchored-fusion's one-fourth limit
    is judged
  repo: https://github.com/ndaniel/fusioncatcher
- name: Arriba
  role: Competing bulk RNA-seq fusion detection method (v2.4.0) run on the same simulated
    fastq inputs
  repo: https://github.com/suhrig/arriba
- name: BWA-MEM
  role: Alignment tool used by Anchored-fusion to map paired-end reads to the anchored
    gene reference and the genome
- name: BLAT
  role: Performs local alignment on candidate chimeric reads to predict the precise
    fusion location
- name: samtools
  role: Required (>=1.16) for alignment post-processing in the Anchored-fusion pipeline
- name: bedtools
  role: Required (>=v2.30.0) for interval operations in the Anchored-fusion pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:b73b67609ddc72d3d1800a704f78cb34da774a6a82a2d07c344d0bd6aac387bd
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
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# simulated-fusion-read-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a targeted fusion detector (Anchored-fusion) against bulk RNA-seq fusion callers (STAR-Fusion, FusionInspector, FusionCatcher, Arriba) on simulated fusion transcripts by tallying spanning/split reads and the minimum read coverage at which each tool detects events such as BCR-ABL1, CIC-DUX4, and DUX4-IGH.

## When to use

Use when comparing fusion gene detection sensitivity across tools on simulated paired-end RNA-seq reads, especially to establish minimum detection coverage for specific fusion genes like BCR-ABL1 or CIC-DUX4.

## When NOT to use

- The data are single-end RNA-seq, whole-genome, or whole-exome sequencing data — Anchored-fusion only supports paired-end RNA-seq
- The goal is de novo discovery of fusions that do not include a user-specified anchor/target gene — Anchored-fusion only reports fusions containing the anchored gene
- Detecting fusions without a known target gene of interest, since Anchored-fusion requires the user to supply the reference sequence of the target gene

## Inputs

- Simulated paired-end RNA-seq fastq.gz files containing fusion transcripts (combined with GM12878 RNA-seq background reads)
- Target gene transcript fasta (e.g., BCR-ABL1, CIC-DUX4, DUX4-IGH) downloaded via NCBI Gene
- Genome fasta (hg38 or hg19) from GENCODE or Ensembl
- Compatible gene annotation gtf matching the genome fasta version

## Outputs

- Per-tool spanning read and split read counts supporting each simulated fusion event
- Minimum read coverage at which each tool detects each fusion gene
- Compiled benchmark table (simulated_fusion_benchmark_results.csv) reproducing Figure 3A/3B

## How to apply

Prepare references by downloading the target gene transcript fasta via NCBI Gene and a compatible hg38/hg19 genome fasta plus gene annotation gtf. Run Anchored-fusion with 'python Anchored_Fusion.py', which maps paired-end reads to the anchored gene with BWA (>=0.7.17) plus samtools (>=1.16), performs local alignment of candidate chimeric reads with BLAT (>=v.35), and uses bedtools (>=v2.30.0) for interval operations; optionally skip or retrain the HVLD deep learning false-positive filter using '--not_filter_false_positive' or '--not_train_filter_model'. Run the competing bulk RNA-seq detectors STAR-Fusion (v1.12.0), FusionInspector (v2.8.0), FusionCatcher (v1.30), and Arriba (v2.4.0) on the same simulated paired-end fastq inputs. For each tool, tally spanning reads and split reads supporting each simulated fusion event, then determine the minimum read coverage at which each tool detects BCR-ABL1 and CIC-DUX4 (Anchored-fusion should be best, i.e., detect at the lowest coverage) and DUX4-IGH (Anchored-fusion should detect at one-fourth of FusionCatcher's coverage limit). Compile per-tool counts and coverage limits into a results table (e.g., simulated_fusion_benchmark_results.csv) matching Figure 3A/3B of the paper.

## Related tools

- **Anchored-fusion** (Targeted fusion detector under benchmark; anchors a gene of interest and recovers non-unique short-read matches, with optional HVLD deep learning false-positive filtering) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **STAR-Fusion** (Competing bulk RNA-seq fusion detection method (v1.12.0) run on the same simulated fastq inputs) — https://github.com/STAR-Fusion
- **FusionInspector** (Competing bulk RNA-seq fusion detection method (v2.8.0) run on the same simulated fastq inputs) — https://github.com/FusionInspector/FusionInspector
- **FusionCatcher** (Competing bulk RNA-seq fusion detection method (v1.30); its coverage limit for DUX4-IGH is the reference against which Anchored-fusion's one-fourth limit is judged) — https://github.com/ndaniel/fusioncatcher
- **Arriba** (Competing bulk RNA-seq fusion detection method (v2.4.0) run on the same simulated fastq inputs) — https://github.com/suhrig/arriba
- **BWA-MEM** (Alignment tool used by Anchored-fusion to map paired-end reads to the anchored gene reference and the genome)
- **BLAT** (Performs local alignment on candidate chimeric reads to predict the precise fusion location)
- **samtools** (Required (>=1.16) for alignment post-processing in the Anchored-fusion pipeline)
- **bedtools** (Required (>=v2.30.0) for interval operations in the Anchored-fusion pipeline)

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=target_gene.fasta --fastq1=test_sample_1.fastq.gz --fastq2=test_sample_1.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- The reproduced table must show Anchored-fusion with the highest sensitivity (lowest detection coverage) for BCR-ABL1 and CIC-DUX4 among all methods tested
- DUX4-IGH detection coverage for Anchored-fusion must equal one-fourth of FusionCatcher's coverage limit
- Per-tool counts of spanning reads and split reads supporting each simulated fusion event must be tallied and consistent across simulated fastq replicates
- All four competing tools (STAR-Fusion v1.12.0, FusionInspector v2.8.0, FusionCatcher v1.30, Arriba v2.4.0) must be run on identical simulated paired-end fastq inputs
- Reference fasta and gtf versions must be compatible (hg38/hg19) to avoid mapping artifacts

## Limitations

- Fusion genes that do not include the user-provided target gene cannot be detected by Anchored-fusion
- Anchored-fusion cannot be used for single-end RNA-seq, whole-genome, or whole-exome sequencing data
- For small input datasets, users must rely on default HVLD model parameters from GitHub rather than retrained parameters, which may deviate
- Using the pre-trained filter model ('--not_train_filter_model') may yield results with some deviations since it was not trained on the user's input data
- The value of the loss-balancing hyperparameter alpha in the HVLD loss function is not specified

## Evidence

- [results] Among all methods tested on simulated fusion transcripts, Anchored-fusion demonstrated the best detection capability for the BCR-ABL1 and CIC-DUX4 fusion genes: [evidence span withheld — anchor status: not_found]
