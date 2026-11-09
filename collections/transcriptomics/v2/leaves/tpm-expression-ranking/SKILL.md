---
name: tpm-expression-ranking
description: Use when Ig-transcript contigs from de novo assembly must be expression-quantified
  and TPM-ranked to mark the dominant clone, e.g. for mCLL/uCLL classification from
  RNA-seq.
when_to_use_negative:
- The input BAM is not aligned to the hg38 (GRCh38) genome build, coordinate-sorted
  and indexed, since CRIS upstream steps will not produce valid Ig transcripts to
  rank
- The goal is SHM genotyping directly from genomic/PCR data rather than from Trinity-assembled
  RNA-seq Ig transcripts
- A pre-existing clonotype table already provides expression-ranked clones, making
  re-quantification redundant
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Salmon
  role: Quantifies Ig-transcript abundances with k-mer size 31 bp, producing the TPM
    values used for ranking (v0.8.1)
- name: seqkit
  role: Sequence handling for sorting and writing the TPM-ranked FASTA (v0.12.0)
- name: BLAST
  role: blastn (v2.9.0) against the custom IMGT IGHV reference (E-value cutoff 20)
    upstream of quantification, and BLAST-based alignment for SHM enumeration downstream
- name: CRIS
  role: Pipeline orchestrating the TPM ranking and writing the sorted table/FASTA
    outputs
  repo: https://github.com/Rashedul/CRIS
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:e36e0d8cba2a9bec3a24cd73a5a6e2733adc385b3caf66c238625e88974e7e83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Rashedul/CRIS
  tools:
  - BLAST
  - CRIS
  - Salmon
  - seqkit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# tpm-expression-ranking

## Summary

Ranks Trinity-assembled immunoglobulin transcripts by expression using Salmon TPM values so that the highest-TPM transcript is flagged as the dominant B-cell clone in the CRIS RNA-seq CLL IGHV typing workflow.

## When to use

Use when Ig-transcript contigs from de novo assembly must be expression-quantified and TPM-ranked to mark the dominant clone, e.g. for mCLL/uCLL classification from RNA-seq.

## When NOT to use

- The input BAM is not aligned to the hg38 (GRCh38) genome build, coordinate-sorted and indexed, since CRIS upstream steps will not produce valid Ig transcripts to rank
- The goal is SHM genotyping directly from genomic/PCR data rather than from Trinity-assembled RNA-seq Ig transcripts
- A pre-existing clonotype table already provides expression-ranked clones, making re-quantification redundant

## Inputs

- Trinity (v2.1.1)-assembled Ig-transcript FASTA, blastn-filtered against the custom IMGT IGHV database (E-value cutoff 20)
- Ig-transcript sequence-ID table from the upstream assembly step
- RNA-seq reads extracted from a coordinate-sorted, indexed BAM aligned to hg38 (GRCh38) with BWA

## Outputs

- Ig-transcripts.sortedbyTPM.txt: Ig-transcript sequence IDs sorted by expression (TPM) values
- Ig-transcripts.sortedbyTPM.fasta: Ig-transcript FASTA sequences sorted by expression (TPM) values
- Dominant-clone flag assigned to the highest-TPM productive Ig transcript

## How to apply

Load the Trinity-assembled Ig-transcript FASTA that was blastn (BLAST v2.9.0) filtered against the custom IMGT IGHV reference at an expectation value cutoff of 20, so only candidate Ig transcripts are quantified. Quantify these transcripts with Salmon (v0.8.1) using a k-mer size of 31 bp, then rank them by TPM (transcripts per million). Write two consistent outputs: a TPM-sorted table of Ig-transcript sequence IDs and a TPM-sorted FASTA of the corresponding sequences. Flag the highest-TPM productive transcript as the dominant clone, which becomes the transcript passed to the IgBLAST SHM/mCLL-uCLL step (identity < 98% is mutated). Use seqkit (v0.12.0) for sequence handling; CRIS requires 4 threads and 16G RAM by default.

## Related tools

- **Salmon** (Quantifies Ig-transcript abundances with k-mer size 31 bp, producing the TPM values used for ranking (v0.8.1))
- **seqkit** (Sequence handling for sorting and writing the TPM-ranked FASTA (v0.12.0))
- **BLAST** (blastn (v2.9.0) against the custom IMGT IGHV reference (E-value cutoff 20) upstream of quantification, and BLAST-based alignment for SHM enumeration downstream)
- **CRIS** (Pipeline orchestrating the TPM ranking and writing the sorted table/FASTA outputs) — https://github.com/Rashedul/CRIS

## Examples

```
bash CRIS.sh -inbam SRR1814049_test.bam -outdir /fullPath/ -threads 4 -memory 16G
```

## Evaluation signals

- The TPM-sorted table exists and contains Ig-transcript sequence IDs ordered by expression (TPM) values
- The TPM-sorted FASTA sequence order matches the ID order in the TPM-sorted table
- For the supplied test dataset SRR1814049_test.bam, the dominant-clone call is consistent with the reported IGHV3-74 assignment at 94.6 percent identity
- The highest-TPM transcript is flagged as the dominant clone and its percent identity (94.6, below the 98% cutoff) yields an mCLL/mutated call in the downstream IgBLAST step

## Limitations

- Ranking depends on the upstream blastn filtering against IMGT IGHV at expectation value cutoff 20; transcripts missed at that step cannot be ranked
- TPM ranking alone marks the dominant clone; SHM status still requires the separate IgBLAST step against the germline V/D/J database with the 98% identity cutoff
- CRIS requires at least 25M sequence reads per sample and a minimum of 16GB RAM and 4 threads; inputs must be ribodepleted or polyA-selected RNA-seq aligned to hg38

## Evidence

- [methods] The Ig transcripts resultant from the blastn filtering were quantified with Salmon (v0.8.1) using a k-mer of 31 bp, and the transcript with the highest TPM (transcripts per million) value was marked as the dominant clone.: [evidence span withheld — anchor status: not_found]
- [readme] Contains Ig-transcript sequence IDs and sorted by expression (TPM) values.: "Contains Ig-transcript sequence IDs and sorted by expression (TPM) values."
- [readme] Contains Ig-transcript fasta sequences sorted by expression (TPM) values.: "Contains Ig-transcript fasta sequences sorted by expression (TPM) values. "
- [methods] Rank Ig transcripts by TPM and write both the TPM-sorted table and the TPM-sorted fasta, flagging the highest-TPM productive transcript as the dominant clone.: [evidence span withheld — anchor status: not_found]
- [readme] Both ribodepleted and polyA selected RNA-seq datasets are appropriate with a minimum of 25M sequence reads per sample.: "Both ribodepleted and polyA selected RNA-seq datasets are appropriate with a minimum of 25M sequence reads per sample."
