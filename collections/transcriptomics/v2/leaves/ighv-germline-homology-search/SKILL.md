---
name: ighv-germline-homology-search
description: Use when you have de novo-assembled transcript contigs (e.g., Trinity
  FASTA) and need to find which contigs are immunoglobulin IGHV-derived and which
  expressed Ig transcript is the dominant clone by TPM.
when_to_use_negative:
- Input reads have not been assembled — this step operates on Trinity-assembled contigs,
  not raw FASTQ or BAM alignments
- You need full IGHV-D-J reconstruction or SHM classification itself; this skill only
  performs germline homology identification and TPM-based dominance ranking (downstream
  steps use IgBLAST v1.14.0)
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_2830
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: BLAST
  role: blastn (v2.9.0) against the custom IMGT IGHV database with E-value cutoff
    20 to identify Ig transcripts among assembled contigs
  repo: https://github.com/Rashedul/CRIS
- name: Salmon
  role: quantifies blastn-filtered Ig transcripts using k-mer 31, producing TPM values
    used to rank and mark the dominant clone
  repo: https://github.com/Rashedul/CRIS
- name: seqkit
  role: v0.12.0 used within CRIS for FASTA sequence manipulation of the sorted Ig
    transcripts
  repo: https://github.com/Rashedul/CRIS
- name: CRIS
  role: pipeline that wraps the blastn IMGT filtering and Salmon TPM quantification
    steps
  repo: https://github.com/Rashedul/CRIS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:bdfcef6869915595f77ff38b7f5fffd1604c883b640a1322c464466ea8875f04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Rashedul/CRIS
  tools:
  - BLAST
  - CRIS
  - Salmon
  - seqkit
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_2830
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# ighv-germline-homology-search

## Summary

Identify immunoglobulin transcripts among Trinity-assembled contigs by blastn against a custom IMGT IGHV reference database (BLAST v2.9.0, E-value cutoff 20), then quantify the retained Ig transcripts with Salmon (v0.8.1, k-mer 31) and rank by TPM to flag the dominant clone.

## When to use

Use when you have de novo-assembled transcript contigs (e.g., Trinity FASTA) and need to find which contigs are immunoglobulin IGHV-derived and which expressed Ig transcript is the dominant clone by TPM.

## When NOT to use

- Input reads have not been assembled — this step operates on Trinity-assembled contigs, not raw FASTQ or BAM alignments
- You need full IGHV-D-J reconstruction or SHM classification itself; this skill only performs germline homology identification and TPM-based dominance ranking (downstream steps use IgBLAST v1.14.0)

## Inputs

- Trinity-assembled immunoglobulin transcript FASTA (from CRIS upstream assembly, e.g., trinity v2.1.1)
- Custom IMGT IGHV germline reference database for blastn

## Outputs

- Ig-transcript sequence IDs sorted by expression (TPM) values (e.g., SRR1814049_test.bam.ig-transcripts.sortedbyTPM.txt)
- Ig-transcript FASTA sequences sorted by expression (TPM) values (e.g., SRR1814049_test.bam.ig-transcripts.sortedbyTPM.fasta)
- Highest-TPM productive transcript flagged as the dominant clone, with its top germline IGHV hit and percent identity

## How to apply

Load the Trinity-assembled Ig transcript FASTA from the upstream de novo assembly step of the CRIS pipeline. Run blastn (BLAST v2.9.0) against the custom IMGT IGHV reference database using an expectation value cutoff of 20 to identify Ig transcripts and record their top germline IGHV hits. Quantify the blastn-filtered Ig transcripts with Salmon v0.8.1 using a k-mer size of 31 bp. Rank transcripts by TPM and write both a TPM-sorted table and a TPM-sorted FASTA, flagging the highest-TPM productive transcript as the dominant clone; this dominant transcript then feeds downstream SHM/percent-identity classification.

## Related tools

- **BLAST** (blastn (v2.9.0) against the custom IMGT IGHV database with E-value cutoff 20 to identify Ig transcripts among assembled contigs) — https://github.com/Rashedul/CRIS
- **Salmon** (quantifies blastn-filtered Ig transcripts using k-mer 31, producing TPM values used to rank and mark the dominant clone) — https://github.com/Rashedul/CRIS
- **seqkit** (v0.12.0 used within CRIS for FASTA sequence manipulation of the sorted Ig transcripts) — https://github.com/Rashedul/CRIS
- **CRIS** (pipeline that wraps the blastn IMGT filtering and Salmon TPM quantification steps) — https://github.com/Rashedul/CRIS

## Examples

```
bash CRIS.sh -inbam SRR1814049_test.bam -outdir /fullPath/ -threads 4 -memory 16G
```

## Evaluation signals

- TPM-sorted table exists and Ig-transcript sequence IDs are ordered by decreasing TPM values
- TPM-sorted FASTA contains the same Ig transcripts, ordered consistently with the TPM table
- For the supplied test dataset, the dominant-clone call matches the expected IGHV3-74 assignment at 94.6 percent identity
- Every transcript in the sorted outputs has a blastn hit against the IMGT IGHV database at E-value <= 20
- Salmon was run with k-mer size 31 on the blastn-filtered Ig transcript FASTA only

## Limitations

- Requires a minimum of 25M sequence reads per sample (ribodepleted or polyA-selected RNA-seq) and 16GB RAM / 4 threads
- Input BAM must be aligned to hg38 (GRCh38) using BWA, coordinate-sorted (SAMBAMBA sort) and indexed before the upstream assembly feeding this step
- Salmon version is pinned to v0.8.1 and BLAST to v2.9.0 in the CRIS dependencies; other versions are untested in this pipeline

## Evidence

- [results] The Ig transcripts resultant from the blastn filtering were quantified with Salmon (v0.8.1) using a k-mer of 31 bp, and the transcript with the highest TPM: [evidence span withheld — anchor status: not_found]
- [other] Run blastn (BLAST v2.9.0) against the custom IMGT IGHV reference database using an expectation value cutoff of 20 to identify Ig-transcripts: [evidence span withheld — anchor status: not_found]
- [readme] Contains Ig-transcript sequence IDs and sorted by expression (TPM) values.: "Contains Ig-transcript sequence IDs and sorted by expression (TPM) values."
- [methods] Test run expected IGHV gene: IGHV3-74 with percent identity 94.6: [evidence span withheld — anchor status: not_found]
- [readme] Dependencies: ```picard (v2.20.3), trinity (v2.1.1), blast (v2.9.0), seqkit (v0.12.0), sambamba (v0.7.0), salmon (v0.8.1), igblast (v1.14.0), jellyfish (v2.2.10)```: "Dependencies: ```picard (v2.20.3), trinity (v2.1.1), blast (v2.9.0), seqkit (v0.12.0), sambamba (v0.7.0), salmon (v0.8.1), igblast (v1.14.0), jellyfish (v2.2.10)```"
