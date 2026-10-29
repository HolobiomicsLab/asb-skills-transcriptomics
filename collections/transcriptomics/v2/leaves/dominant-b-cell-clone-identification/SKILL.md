---
name: dominant-b-cell-clone-identification
description: Use when Trinity-assembled contigs from an RNA-seq sample (e.g. CLL patient
  BAM aligned to hg38) must be filtered for Ig transcripts and the highest-TPM productive
  transcript marked as the dominant clone.
when_to_use_negative:
- Input is raw RNA-seq reads or an unaligned BAM rather than Trinity-assembled Ig
  transcript contigs from the upstream CRIS assembly step
- The BAM is not aligned to hg38 (GRCh38) with BWA, coordinate-sorted and indexed,
  as required by CRIS
- The goal is full germline-aligned SHM/SNV enumeration rather than dominant clone
  identification by expression ranking
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0160
tools:
- name: BLAST v2.9.0
  role: blastn search of Trinity-assembled Ig transcript contigs against the custom
    IMGT IGHV database (E-value cutoff 20) to identify Ig transcripts and their top
    germline IGHV hits
  repo: https://github.com/Rashedul/CRIS
- name: Salmon v0.8.1
  role: quantifies blastn-filtered Ig transcripts using a k-mer size of 31 bp to produce
    TPM values used to rank clones
  repo: https://github.com/Rashedul/CRIS
- name: Trinity v2.1.1
  role: upstream de novo assembly step producing the Ig transcript contigs that this
    skill filters and ranks
  repo: https://github.com/Rashedul/CRIS
- name: CRIS
  role: enclosing pipeline (Docker or bash) that orchestrates the blastn/IMGT filtering
    and Salmon TPM quantification and writes the TPM-sorted outputs
  repo: https://github.com/Rashedul/CRIS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:105258f0e05a76bf9e21f239f653f7002e2abfb4cffc7d8b42e58da5c40da253
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Rashedul/CRIS
  tools:
  - BLAST v2.9.0
  - CRIS
  - Salmon v0.8.1
  - Trinity v2.1.1
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# dominant-b-cell-clone-identification

## Summary

Identifies immunoglobulin transcripts among Trinity-assembled contigs and flags the dominant clone by quantifying them with Salmon (v0.8.1, k-mer 31) and ranking by TPM. Part of the CRIS pipeline for reconstructing IGHV-D-J sequences from RNA-seq.

## When to use

Use when Trinity-assembled contigs from an RNA-seq sample (e.g. CLL patient BAM aligned to hg38) must be filtered for Ig transcripts and the highest-TPM productive transcript marked as the dominant clone.

## When NOT to use

- Input is raw RNA-seq reads or an unaligned BAM rather than Trinity-assembled Ig transcript contigs from the upstream CRIS assembly step
- The BAM is not aligned to hg38 (GRCh38) with BWA, coordinate-sorted and indexed, as required by CRIS
- The goal is full germline-aligned SHM/SNV enumeration rather than dominant clone identification by expression ranking

## Inputs

- Trinity-assembled Ig transcript FASTA (contigs from upstream CRIS assembly step)
- Custom IMGT IGHV germline reference database for blastn

## Outputs

- Ig-transcript sequence IDs sorted by expression (TPM) values (e.g. SRR1814049_test.bam.ig-transcripts.sortedbyTPM.txt)
- Ig-transcript FASTA sequences sorted by expression (TPM) values (e.g. SRR1814049_test.bam.ig-transcripts.sortedbyTPM.fasta)
- Dominant clone designation (highest-TPM transcript) with its top germline IGHV hit and percent identity

## How to apply

Load the Trinity-assembled Ig transcript FASTA produced by the upstream CRIS de novo assembly step and run blastn (BLAST v2.9.0) against the custom IMGT IGHV reference database with an expectation value cutoff of 20 to retain Ig transcripts and their top germline IGHV hits. Quantify the surviving Ig transcripts with Salmon (v0.8.1) using a k-mer size of 31 bp to obtain TPM values. Rank the transcripts by TPM and write both a TPM-sorted table (.ig-transcripts.sortedbyTPM.txt) and a TPM-sorted FASTA (.ig-transcripts.sortedbyTPM.fasta). The transcript with the highest TPM is flagged as the dominant clone; its identity and percent identity to the germline are subsequently reported via IgBLAST-style output (e.g. IGHV3-74 at 94.6% for the supplied test dataset).

## Related tools

- **BLAST v2.9.0** (blastn search of Trinity-assembled Ig transcript contigs against the custom IMGT IGHV database (E-value cutoff 20) to identify Ig transcripts and their top germline IGHV hits) — https://github.com/Rashedul/CRIS
- **Salmon v0.8.1** (quantifies blastn-filtered Ig transcripts using a k-mer size of 31 bp to produce TPM values used to rank clones) — https://github.com/Rashedul/CRIS
- **Trinity v2.1.1** (upstream de novo assembly step producing the Ig transcript contigs that this skill filters and ranks) — https://github.com/Rashedul/CRIS
- **CRIS** (enclosing pipeline (Docker or bash) that orchestrates the blastn/IMGT filtering and Salmon TPM quantification and writes the TPM-sorted outputs) — https://github.com/Rashedul/CRIS

## Examples

```
bash CRIS.sh -inbam SRR1814049_test.bam -outdir /fullPath/ -threads 4 -memory 16G
```

## Evaluation signals

- The TPM-sorted table (.ig-transcripts.sortedbyTPM.txt) exists and contains Ig-transcript sequence IDs ordered by TPM expression values
- The TPM-sorted FASTA (.ig-transcripts.sortedbyTPM.fasta) contains Ig-transcript sequences in the same TPM order as the table
- For the supplied test BAM SRR1814049_test.bam, the dominant-clone call is consistent with the reported IGHV3-74 assignment at 94.6% identity
- The blastn step used the IMGT IGHV database with E-value cutoff 20, and Salmon used k-mer 31; deviating from these parameters invalidates comparability with the published run

## Limitations

- Requires the upstream CRIS steps: BAM aligned to hg38 (GRCh38) using BWA, coordinate-sorted and indexed, with Trinity v2.1.1 assembly beforehand
- Version-sensitive: Salmon v0.8.1 (k=31) and BLAST v2.9.0 with E-value cutoff 20 are the parameters under which the test-case call (IGHV3-74, 94.6% identity) was validated
- Depends on the custom IMGT IGHV reference database for germline assignment; results are IGHV-chain-specific, not a general repertoire profiler

## Evidence

- [abstract] performs assembly of Ig transcripts and aligns the resulting contigs to reference Ig sequences: "performs assembly of Ig transcripts and aligns the resulting contigs to reference Ig sequences"
- [methods] blastn/IMGT filtering and Salmon TPM quantification with k-mer 31: [evidence span withheld — anchor status: not_found]
- [readme] TPM-sorted output table description: "Contains Ig-transcript sequence IDs and sorted by expression (TPM) values."
- [readme] CRIS dependency versions including blast and salmon: "picard (v2.20.3), trinity (v2.1.1), blast (v2.9.0), seqkit (v0.12.0), sambamba (v0.7.0), salmon (v0.8.1)"
- [methods] Expected test run result anchoring the validation signal: [evidence span withheld — anchor status: not_found]
