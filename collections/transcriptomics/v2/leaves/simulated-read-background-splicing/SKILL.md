---
name: simulated-read-background-splicing
description: Use when benchmarking a fusion-gene detection tool and you need test
  FASTQs containing known fusion junctions at controlled expression/coverage levels
  embedded in realistic sequencing background.
when_to_use_negative:
- You need single-end RNA-seq test data — the simulated reads and downstream detection
  (e.g., Anchored-fusion) are paired-end only
- The goal is de novo discovery of unknown fusions rather than benchmarking detection
  of known fusions at controlled abundance
- Whole-genome or whole-exome sequencing data rather than transcriptomic RNA-seq background
  is required
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: wgsim
  role: Simulates paired-end reads from the six fusion transcript references at seven
    doubling coverage levels (2x-128x)
- name: Anchored-fusion
  role: Consumer of the simulated test datasets via --fastq1/--fastq2 for sensitivity
    benchmarking
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:34a374001f51b6ff8e604860816b39d69398125f97b24cad05a2817cafdafb2f
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
  - wgsim
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# simulated-read-background-splicing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs realistic RNA-seq benchmark test datasets by simulating fusion-transcript reads with wgsim at graded coverage levels and splicing them into real background RNA-seq reads (GM12878), enabling sensitivity benchmarking of fusion-detection tools like Anchored-fusion.

## When to use

Use when benchmarking a fusion-gene detection tool and you need test FASTQs containing known fusion junctions at controlled expression/coverage levels embedded in realistic sequencing background.

## When NOT to use

- You need single-end RNA-seq test data — the simulated reads and downstream detection (e.g., Anchored-fusion) are paired-end only
- The goal is de novo discovery of unknown fusions rather than benchmarking detection of known fusions at controlled abundance
- Whole-genome or whole-exome sequencing data rather than transcriptomic RNA-seq background is required

## Inputs

- Transcript FASTA sequences of the twelve fusion partner genes from NCBI Gene
- Six in-silico fusion transcript references (partner pairs joined at breakpoint junctions)
- Genome FASTA from GENCODE or Ensembl (hg38 or hg19)
- Matching gene annotation GTF compatible with the genome reference version
- Paired-end RNA-seq reads from the GM12878 cell line as background

## Outputs

- 6 gene pairs x 7 coverage levels = paired gzip-compressed FASTQ test datasets (R1/R2)
- Test samples containing reads spanning the simulated fusion junction embedded in GM12878 background reads

## How to apply

Build six fusion transcript references by downloading partner-gene transcript FASTAs (BCR-ABL1, CIC-DUX4, DUX4-IGH, TMPRSS2-ERG, EML4-ALK, TPM3-NTRK1) from NCBI Gene and joining each pair at its breakpoint junction. Obtain a matching genome FASTA (GENCODE/Ensembl, hg38 or hg19) and compatible gene annotation GTF. Run wgsim on each fusion transcript reference to generate paired-end reads at seven coverage levels doubling from 2x up to 128x (base coverage 128 down to 2). Combine each simulated read set with paired-end RNA-seq reads from the GM12878 cell line as background, then write the results as paired gzip-compressed FASTQ (R1/R2) files in the format expected by the detection tool's --fastq1/--fastq2 arguments.

## Related tools

- **wgsim** (Simulates paired-end reads from the six fusion transcript references at seven doubling coverage levels (2x-128x))
- **Anchored-fusion** (Consumer of the simulated test datasets via --fastq1/--fastq2 for sensitivity benchmarking) — https://github.com/ShenLab-Genomics/Anchored-Fusion

## Examples

```
wgsim -d 128 fusion_ref_BCR_ABL1.fasta sim_r1.fastq sim_r2.fastq && gzip sim_r1.fastq sim_r2.fastq && cat sim_r1.fastq.gz GM12878_R1.fastq.gz > test_sample_1.fastq.gz
```

## Evaluation signals

- All 6 gene pairs x 7 coverage levels produced paired fastq.gz test datasets
- Each test dataset contains reads spanning the simulated fusion junction
- Simulated fusion reads are embedded in GM12878 background RNA-seq reads
- Coverage levels double correctly from 2x up to 128x (base coverage 128 down to 2)
- FASTQ pairing (R1/R2) is intact and the files are accepted by Anchored_Fusion.py's --fastq1/--fastq2 arguments

## Limitations

- Simulated fusions are restricted to the six constructed partner pairs (BCR-ABL1, CIC-DUX4, DUX4-IGH, TMPRSS2-ERG, EML4-ALK, TPM3-NTRK1); other fusions are not represented
- Coverage is limited to the seven simulated levels (2x-128x); sensitivity outside this range is untested
- The GM12878 background may not reflect the sequence context (e.g., homology, GC content) of other tissues or cancer types

## Evidence

- [other] The simulated benchmark datasets were constructed by using wgsim to generate fusion transcripts at seven expression (coverage) levels ranging from a base coverage of 128 down to 2, and then adding these simulated fusion transcripts into the RNA-seq data of the GM12878 cell line to serve as the test data.: [evidence span withheld — anchor status: not_found]
