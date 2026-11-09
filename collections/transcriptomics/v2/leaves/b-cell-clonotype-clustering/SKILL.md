---
name: b-cell-clonotype-clustering
description: Use when RNA-seq-derived Ig transcript contigs (e.g. Trinity-assembled
  from a BWA/hg38-aligned BAM) need IGHV germline assignment, percent-identity-based
  SHM calls, and clonotype grouping to classify CLL samples as mCLL/uCLL.
when_to_use_negative:
- Input is raw reads or a BAM not aligned to hg38 (GRCh38), coordinate-sorted and
  indexed — CRIS requires that preprocessed input before this step
- Goal is full BCR repertoire profiling from single-cell or dedicated AIRR-seq data
  rather than bulk RNA-seq IGHV SHM-status classification of CLL samples
- Fewer than 25M sequence reads per sample or non-ribodepleted/polyA-incompatible
  libraries — README states a minimum of 25M reads for both ribodepleted and polyA
  selected RNA-seq
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_2830
- http://edamontology.org/topic_3308
tools:
- name: CRIS
  role: Pipeline that extracts Ig-locus reads, assembles Ig transcripts, and runs
    the IgBLAST-based SHM/clonotype step
  repo: https://github.com/Rashedul/CRIS
- name: IgBLAST
  role: Aligns Ig transcripts against germline V/D/J references to return top IGHV
    hit, percent identity, and clonotype grouping (v1.14.0)
- name: BLAST
  role: v2.9.0-based alignment used to enumerate SHMs against the top germline IGHV
    hit
- name: seqkit
  role: Sequence handling for TPM-sorted Ig-transcript FASTA files (v0.12.0)
- name: trinity
  role: De novo assembly of Ig transcripts feeding this step (v2.1.1)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:1b0cbfea26a21a9d752fc92e6442ee3d204bc9aa0b365a910e6f97bfb2f5d776
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Rashedul/CRIS
  tools:
  - BLAST
  - CRIS
  - IgBLAST
  - seqkit
  - trinity
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_2830
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# b-cell-clonotype-clustering

## Summary

CRIS's SHM/clonotyping step queries assembled Ig transcripts against IMGT germline V/D/J references with IgBLAST, clusters transcripts sharing the same V-D-J rearrangement into clonotypes ranked by TPM, and calls mutated (mCLL, identity <98%) versus unmutated (uCLL, identity >=98%) IGHV status.

## When to use

Use when RNA-seq-derived Ig transcript contigs (e.g. Trinity-assembled from a BWA/hg38-aligned BAM) need IGHV germline assignment, percent-identity-based SHM calls, and clonotype grouping to classify CLL samples as mCLL/uCLL.

## When NOT to use

- Input is raw reads or a BAM not aligned to hg38 (GRCh38), coordinate-sorted and indexed — CRIS requires that preprocessed input before this step
- Goal is full BCR repertoire profiling from single-cell or dedicated AIRR-seq data rather than bulk RNA-seq IGHV SHM-status classification of CLL samples
- Fewer than 25M sequence reads per sample or non-ribodepleted/polyA-incompatible libraries — README states a minimum of 25M reads for both ribodepleted and polyA selected RNA-seq

## Inputs

- Ig-transcript FASTA sorted by TPM expression (SRR1814049_test.bam.ig-transcripts.sortedbyTPM.fasta)
- Transcript-ID table sorted by TPM (SRR1814049_test.bam.ig-transcripts.sortedbyTPM.txt)
- IMGT germline V, D and J reference sequences
- Input BAM aligned to hg38 (GRCh38) with BWA, coordinate-sorted (sambamba) and indexed

## Outputs

- IgBLAST output table with percent identity and alignment between Ig-transcripts and top germline IGHV gene hits (SRR1814049_test.bam.IgBLAST_out.txt)
- Structured SHM record containing dominant IGHV gene, percent identity, and mCLL/uCLL call
- Clonotype table assigning each Ig transcript to its clonotype

## How to apply

Load the TPM-ranked Ig-transcript FASTA and transcript-ID table from the prior assembly step and align each transcript against germline V, D and J references using IgBLAST (v1.14.0) to obtain the top germline IGHV hit and its percent identity; alignment in the CRIS pipeline is BLAST v2.9.0-based, and sequence handling uses seqkit v0.12.0. Enumerate somatic hypermutations by comparing each transcript alignment to its top germline IGHV hit over the entire IGHV gene segment (CRIS's advantage over tools using only a portion of IGHV). Cluster transcripts sharing the same V-D-J rearrangement into clonotypes, prioritizing transcripts by TPM expression ranking. Call SHM status as mCLL when percent identity is <98% and uCLL when >=98%, and record the dominant IGHV gene, percent identity, and SHM call. Validate on the supplied SRR1814049_test.bam, which must reproduce IGHV3-74 with percent identity 94.6 (an mCLL call).

## Related tools

- **CRIS** (Pipeline that extracts Ig-locus reads, assembles Ig transcripts, and runs the IgBLAST-based SHM/clonotype step) — https://github.com/Rashedul/CRIS
- **IgBLAST** (Aligns Ig transcripts against germline V/D/J references to return top IGHV hit, percent identity, and clonotype grouping (v1.14.0))
- **BLAST** (v2.9.0-based alignment used to enumerate SHMs against the top germline IGHV hit)
- **seqkit** (Sequence handling for TPM-sorted Ig-transcript FASTA files (v0.12.0))
- **trinity** (De novo assembly of Ig transcripts feeding this step (v2.1.1))

## Examples

```
bash CRIS.sh -inbam SRR1814049_test.bam -outdir /fullPath/ -threads 4 -memory 16G
```

## Evaluation signals

- Test run on SRR1814049_test.bam must reproduce IGHV gene IGHV3-74 with percent identity 94.6
- Percent identity 94.6 falls below the 98% cutoff, so the SHM call must be mCLL (mutated)
- SRR1814049_test.bam.IgBLAST_out.txt exists and contains percent identity plus alignment between Ig-transcript and top germline IGHV gene hits
- Every Ig transcript in the TPM-sorted FASTA is assigned to exactly one clonotype in the clonotype table
- TPM ordering in the clonotype/record output matches the TPM-sorted transcript table produced by the assembly step

## Limitations

- Requires a minimum of 25M sequence reads per sample; both ribodepleted and polyA selected RNA-seq are supported
- Requires minimum 16GB RAM and 4 threads (Docker defaults: threads=4 memory=16G)
- Input BAM must be aligned to hg38 (GRCh38) using BWA, coordinate-sorted and indexed; other genome builds are not supported
- SHM status is based on the entire IGHV gene segment, so results differ from tools that infer the repertoire using only a portion of the IGHV gene segment

## Evidence

- [other] IgBLAST returns the percent identity of the IGHV segment versus germline alleles and clusters similar transcripts into clonotypes.: [evidence span withheld — anchor status: not_found]
- [readme] Contains the percent identity and alignment between Ig-transcript and top germline IGHV gene hits.: "Contains the percent identity and alignment between Ig-transcript and top germline IGHV gene hits."
- [methods] - IGHV gene: IGHV3-74
- Percent identity: 94.6: "- IGHV gene: IGHV3-74
- Percent identity: 94.6"
- [abstract] CRIS improves on existing tools that infer the B-cell receptor repertoire from RNA-seq data using a portion IGHV gene segment by de novo assembly: "CRIS improves on existing tools that infer the B-cell receptor repertoire from RNA-seq data using a portion IGHV gene segment by de novo assembly"
- [readme] Dependencies: picard (v2.20.3), trinity (v2.1.1), blast (v2.9.0), seqkit (v0.12.0), sambamba (v0.7.0), salmon (v0.8.1), igblast (v1.14.0), jellyfish (v2.2.10): "Dependencies: ```picard (v2.20.3), trinity (v2.1.1), blast (v2.9.0), seqkit (v0.12.0), sambamba (v0.7.0), salmon (v0.8.1), igblast (v1.14.0), jellyfish (v2.2.10)```"
