---
name: fusion-transcript-reference-construction
description: Use when benchmarking a fusion-detection tool (e.g., Anchored-fusion)
  and you need ground-truth test FASTQs containing reads spanning a known fusion junction
  embedded in realistic background RNA-seq data.
when_to_use_negative:
- You need to detect fusions de novo without a predefined target gene — Anchored-fusion
  only reports fusions including the anchored gene
- Input is single-end RNA-seq, whole-genome, or whole-exome sequencing data — Anchored-fusion
  requires paired-end RNA-seq
- You lack a reference sequence for the target gene, since Anchored-fusion requires
  users to provide the target-gene transcript reference
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: wgsim
  role: Simulates paired-end reads from each fusion transcript reference at seven
    coverage levels doubling from 2x up to 128x
- name: Anchored-fusion
  role: Consumer of the constructed test datasets via its --fastq1/--fastq2 paired
    FASTQ input; the reference and annotation files follow its stated input requirements
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:37caebb5e6bb41049c7658073b6309eb25803fc0019a18b02117b9dc1620623b
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# fusion-transcript-reference-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct in-silico fusion-transcript benchmark datasets by joining partner-gene transcript FASTAs (e.g., BCR-ABL1, TMPRSS2-ERG) at their breakpoint junctions, simulating paired-end reads with wgsim across coverage levels (2x–128x), and spiking them into GM12878 RNA-seq background reads.

## When to use

Use when benchmarking a fusion-detection tool (e.g., Anchored-fusion) and you need ground-truth test FASTQs containing reads spanning a known fusion junction embedded in realistic background RNA-seq data.

## When NOT to use

- You need to detect fusions de novo without a predefined target gene — Anchored-fusion only reports fusions including the anchored gene
- Input is single-end RNA-seq, whole-genome, or whole-exome sequencing data — Anchored-fusion requires paired-end RNA-seq
- You lack a reference sequence for the target gene, since Anchored-fusion requires users to provide the target-gene transcript reference

## Inputs

- Transcript FASTA sequences of fusion partner genes (from NCBI Gene)
- Genome FASTA (from GENCODE or Ensembl, hg38 or hg19)
- Compatible gene annotation GTF (hg38 or hg19)
- Paired-end RNA-seq FASTQ from GM12878 as background reads

## Outputs

- Six fusion transcript reference FASTAs joined at breakpoint junctions
- Simulated paired-end fusion reads at seven coverage levels (2x to 128x) via wgsim
- Paired gzip-compressed FASTQ test datasets (R1/R2) with simulated fusion reads embedded in GM12878 background

## How to apply

Download transcript FASTA sequences for each fusion partner gene (twelve genes, six pairs: BCR-ABL1, CIC-DUX4, DUX4-IGH, TMPRSS2-ERG, EML4-ALK, TPM3-NTRK1) via NCBI Gene, and join each partner pair at the breakpoint junction to form six fusion-transcript references. Obtain a genome FASTA from GENCODE or Ensembl plus a matching gene annotation GTF (hg38 or hg19), ensuring reference and annotation versions are compatible. Run wgsim on each fusion transcript reference to simulate paired-end reads at seven coverage levels doubling from 2x up to 128x, then combine each simulated read set with paired-end RNA-seq reads from the GM12878 cell line as background. Write the result as paired gzip-compressed FASTQ files (R1/R2) matching the --fastq1/--fastq2 input format of Anchored_Fusion.py.

## Related tools

- **wgsim** (Simulates paired-end reads from each fusion transcript reference at seven coverage levels doubling from 2x up to 128x)
- **Anchored-fusion** (Consumer of the constructed test datasets via its --fastq1/--fastq2 paired FASTQ input; the reference and annotation files follow its stated input requirements) — https://github.com/ShenLab-Genomics/Anchored-Fusion

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=test/target_gene.fasta --fastq1=test/test_sample_1.fastq.gz --fastq2=test/test_sample_1.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- All 6 gene pairs × 7 coverage levels yield paired fastq.gz test datasets
- Test datasets contain reads spanning the simulated fusion junction embedded in the GM12878 background
- Genome FASTA and GTF annotation versions are compatible (both hg38 or both hg19)
- Coverage levels correctly span 2x to 128x with doubling steps (2, 4, 8, 16, 32, 64, 128)
- Output FASTQs are gzip-compressed and satisfy the Anchored_Fusion.py --fastq1/--fastq2 paired input format

## Limitations

- Simulated fusion reads must be combined with real background (GM12878) to be realistic; purely synthetic datasets may not reflect sequencing artifacts
- Reference FASTA and GTF version mismatches are an explicit pitfall flagged by the authors
- The wgsim simulation models only the tested six fusion pairs; other fusions or breakpoint structures are not covered

## Evidence

- [other] The simulated benchmark datasets were constructed by using wgsim to generate fusion transcripts at seven expression (coverage) levels ranging from a base coverage of 128 down to 2: [evidence span withheld — anchor status: not_found]
