---
name: shm-mutation-enumeration-in-ighv
description: Use when a de novo assembled Ig transcript (FASTA) needs its top germline
  IGHV gene, percent identity, SHM count, and mutated/unmutated (mCLL/uCLL) call determined
  from RNA-seq data.
when_to_use_negative:
- The goal is only partial-IGHV repertoire inference from a portion of the IGHV gene
  segment — CRIS's SHM classification is concordant because it uses the entire reconstructed
  IGHV gene segment
- The input reads are not aligned to the hg38 (GRCh38) genome build and coordinate-sorted/indexed
  as CRIS requires
- Only non-human or non-B-cell RNA-seq data without Ig gene loci reads is available
edam_operation: http://edamontology.org/operation_0447
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3676
tools:
- name: IgBLAST
  role: Aligns assembled Ig transcripts to germline V/D/J references to return the
    top IGHV gene hit, percent identity, and SHM enumeration
- name: BLAST v2.9.0
  role: Alignment engine underlying the transcript-to-germline comparison inside the
    CRIS pipeline
- name: seqkit v0.12.0
  role: Sequence handling for the TPM-ranked Ig-transcript FASTA during clonotype
    clustering
- name: CRIS
  role: Pipeline that orchestrates transcript extraction, assembly, and the SHM/clonotype
    analysis step
  repo: https://github.com/Rashedul/CRIS
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:8ce5108bac81c839fc574eaee96d64f7c0c1f98d6a70840a281322d92b59e650
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Rashedul/CRIS
  tools:
  - BLAST v2.9.0
  - CRIS
  - IgBLAST
  - seqkit
  - seqkit v0.12.0
  edam_operation: http://edamontology.org/operation_0447
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3676
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# shm-mutation-enumeration-in-ighv

## Summary

Enumerates somatic hypermutations (SHMs) and classifies IGHV mutation status from RNA-seq by aligning assembled Ig-transcript contigs to the top germline IGHV hit with IgBLAST, then calling mCLL (<98% identity) versus uCLL (>=98% identity).

## When to use

Use when a de novo assembled Ig transcript (FASTA) needs its top germline IGHV gene, percent identity, SHM count, and mutated/unmutated (mCLL/uCLL) call determined from RNA-seq data.

## When NOT to use

- The goal is only partial-IGHV repertoire inference from a portion of the IGHV gene segment — CRIS's SHM classification is concordant because it uses the entire reconstructed IGHV gene segment
- The input reads are not aligned to the hg38 (GRCh38) genome build and coordinate-sorted/indexed as CRIS requires
- Only non-human or non-B-cell RNA-seq data without Ig gene loci reads is available

## Inputs

- TPM-ranked Ig-transcript FASTA (e.g. SRR1814049_test.bam.ig-transcripts.sortedbyTPM.fasta)
- Ig-transcript ID table sorted by TPM expression (e.g. SRR1814049_test.bam.ig-transcripts.sortedbyTPM.txt)
- IMGT germline V, D and J reference database
- Coordinate-sorted, indexed BAM aligned to hg38 (GRCh38) as upstream CRIS input

## Outputs

- IgBLAST output file (e.g. SRR1814049_test.bam.IgBLAST_out.txt) with percent identity and alignment to top germline IGHV hit
- Structured SHM record: dominant IGHV gene, percent identity, mCLL/uCLL call
- Clonotype table assigning each Ig transcript to its clonotype

## How to apply

Load the TPM-ranked Ig-transcript FASTA and transcript-ID table (e.g. SRR1814049_test.bam.ig-transcripts.sortedbyTPM.fasta) produced by the prior transcript-assembly step of CRIS. Align each transcript against the IMGT germline V, D and J database with IgBLAST v1.14.0 (BLAST v2.9.0-based alignment within CRIS) to obtain the top germline IGHV gene hit and its percent identity. Enumerate somatic hypermutations by comparing each transcript alignment to its top germline IGHV hit, and cluster transcripts sharing the same V-D-J rearrangement into clonotypes prioritized by TPM expression ranking (seqkit v0.12.0 for sequence handling). Call SHM status as mCLL when identity is below the 98% cutoff and uCLL when identity is >=98%, writing a structured record with the dominant IGHV gene, percent identity, and SHM call. The IgBLAST output file (e.g. SRR1814049_test.bam.IgBLAST_out.txt) contains the percent identity and alignment between the Ig-transcript and its top germline IGHV gene hit.

## Related tools

- **IgBLAST** (Aligns assembled Ig transcripts to germline V/D/J references to return the top IGHV gene hit, percent identity, and SHM enumeration)
- **BLAST v2.9.0** (Alignment engine underlying the transcript-to-germline comparison inside the CRIS pipeline)
- **seqkit v0.12.0** (Sequence handling for the TPM-ranked Ig-transcript FASTA during clonotype clustering)
- **CRIS** (Pipeline that orchestrates transcript extraction, assembly, and the SHM/clonotype analysis step) — https://github.com/Rashedul/CRIS

## Examples

```
bash CRIS.sh -inbam SRR1814049_test.bam
```

## Evaluation signals

- Test run on the supplied SRR1814049_test.bam must report IGHV gene IGHV3-74 with percent identity 94.6, i.e. a mutated/mCLL call below the 98% cutoff
- IgBLAST_out.txt exists and contains the percent identity plus alignment between the Ig-transcript and its top germline IGHV gene hit
- Every Ig transcript receives exactly one top germline IGHV gene assignment and a percent-identity value in the [0,100] range
- SHM call is consistent with the 98% identity threshold: identity <98% -> mCLL, identity >=98% -> uCLL
- Clonotype table assigns each transcript sharing the same V-D-J rearrangement to one clonotype, ordered by TPM ranking

## Limitations

- Requires at least 25M sequence reads per sample; only ribodepleted or polyA selected RNA-seq datasets are appropriate
- Minimum 16GB RAM and 4 threads are required; the SHM/classification accuracy was validated only in the context of chronic lymphocytic leukemia against clinical PCR-Sanger classification
- Identity difference versus other tools can arise from reconstruction length — e.g. a partial 226 bp V'DJer assembly gave 89.4% identity vs CRIS's 295/296 bp at 91.2% for US-1422368, so SHM calls are only comparable over the same IGHV span

## Evidence

- [methods] CRIS's SHM/clonotype module queries the Ig transcripts from step 2 against the IMGT germline V, D and J database with IgBLAST: [evidence span withheld — anchor status: not_found]
- [methods] IgBLAST returns the percent identity of the IGHV segment versus germline alleles and clusters similar transcripts into clonotypes.: [evidence span withheld — anchor status: not_found]
- [methods] Call SHM status as mCLL (mutated, identity < 98%) or uCLL (unmutated, identity >= 98%): [evidence span withheld — anchor status: not_found]
- [abstract] The SHM status identified by CRIS using the entire IGHV gene segment is highly concordant with clinical classification in three independent chronic lymphocytic leukemia patient cohorts: "We show that the SHM status identified by CRIS using the entire IGHV gene segment is highly concordant with clinical classification in three independent chronic lymphocytic leukemia patient cohorts"
- [readme] SRR1814049_test.bam.IgBLAST_out.txt Contains the percent identity and alignment between Ig-transcript and top germline IGHV gene hits.: "SRR1814049_test.bam.IgBLAST_out.txt Contains the percent identity and alignment between Ig-transcript and top germline IGHV gene hits."
