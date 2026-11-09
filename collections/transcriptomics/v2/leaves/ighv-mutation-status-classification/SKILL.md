---
name: ighv-mutation-status-classification
description: Use when CLL RNA-seq BAMs (≥25M reads, ribodepleted or polyA) aligned
  to hg38/GRCh38 with BWA need an IGHV mutation status call, and clinical Sanger genotyping
  is unavailable or should be cross-checked from transcriptome data.
when_to_use_negative:
- Input BAMs are not aligned to hg38 (GRCh38) — CRIS requires BWA-aligned hg38, coordinate-sorted
  BAMs; other builds or unsorted files are unsupported
- Samples have fewer than 25M RNA-seq reads per sample, insufficient for de novo assembly
  of the IGHV transcript
- Goal is full B-cell receptor repertoire profiling from mixed samples rather than
  CLL IGHV hypermutation status per patient
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_2830
- http://edamontology.org/topic_3170
tools:
- name: CRIS
  role: Pipeline that reconstructs IGHV from RNA-seq and predicts hypermutation (mCLL/uCLL)
    status
  repo: https://github.com/Rashedul/CRIS
- name: trinity v2.1.1
  role: De novo assembly of Ig transcripts extracted from the input BAM
- name: igblast v1.14.0
  role: Aligns assembled Ig transcripts to IMGT germline V/D/J sequences to obtain
    top IGHV hit and percent identity
- name: blast v2.9.0
  role: Alignment of contigs to reference Ig sequences used to enumerate somatic hypermutations
- name: seqkit v0.12.0
  role: Sequence handling for TPM-ranked Ig-transcript FASTA/tables
- name: salmon v0.8.1
  role: Quantifies Ig transcripts to produce TPM rankings used to select the dominant
    clone
- name: sambamba v0.7.0
  role: Coordinate-sorts and validates input BAMs
- name: picard v2.20.3
  role: BAM processing dependency of the CRIS pipeline
- name: jellyfish v2.2.10
  role: k-mer counting dependency of the CRIS pipeline
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:c1b4061e071b8369ff6d0f780e6d7fc622568f4f1bf38a14de47a5deeb843187
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Rashedul/CRIS
  tools:
  - CRIS
  - Docker
  - blast
  - blast v2.9.0
  - igblast v1.14.0
  - jellyfish
  - jellyfish v2.2.10
  - picard
  - picard v2.20.3
  - salmon
  - salmon v0.8.1
  - sambamba
  - sambamba v0.7.0
  - seqkit
  - seqkit v0.12.0
  - trinity
  - trinity v2.1.1
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_2830
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# ighv-mutation-status-classification

## Summary

Classify chronic lymphocytic leukemia samples as mutated (mCLL) or unmutated (uCLL) by de novo assembling IGHV transcripts from RNA-seq BAMs with CRIS and comparing the dominant clone's IGHV sequence to its top germline hit. Validated against clinical PCR-Sanger IGHV classification with high concordance across three CLL cohorts.

## When to use

Use when CLL RNA-seq BAMs (≥25M reads, ribodepleted or polyA) aligned to hg38/GRCh38 with BWA need an IGHV mutation status call, and clinical Sanger genotyping is unavailable or should be cross-checked from transcriptome data.

## When NOT to use

- Input BAMs are not aligned to hg38 (GRCh38) — CRIS requires BWA-aligned hg38, coordinate-sorted BAMs; other builds or unsorted files are unsupported
- Samples have fewer than 25M RNA-seq reads per sample, insufficient for de novo assembly of the IGHV transcript
- Goal is full B-cell receptor repertoire profiling from mixed samples rather than CLL IGHV hypermutation status per patient

## Inputs

- Coordinate-sorted, indexed BAM aligned to hg38 (GRCh38) with BWA
- RNA-seq dataset with minimum 25M reads per sample (ribodepleted or polyA selected)
- Optional: clinical Sanger-based IGHV mutation classification table for concordance comparison

## Outputs

- <sample>.bam.IgBLAST_out.txt — percent identity and alignment between Ig-transcript and top germline IGHV gene hits
- <sample>.bam.ig-transcripts.sortedbyTPM.txt — Ig-transcript sequence IDs sorted by TPM expression
- <sample>.bam.ig-transcripts.sortedbyTPM.fasta — Ig-transcript FASTA sequences sorted by TPM
- Per-sample mCLL/uCLL call with dominant IGHV gene and percent identity to germline

## How to apply

Run CRIS (bash CRIS.sh -inbam <sample>.bam -outdir <dir>, default threads=4 and memory=16G for Trinity assembly) on each coordinate-sorted, indexed hg38 BAM; the pipeline extracts reads at Ig gene loci, assembles Ig transcripts with Trinity v2.1.1, and aligns contigs to reference Ig sequences with BLAST v2.9.0/IgBLAST v1.14.0. Parse the per-sample IgBLAST output (*.IgBLAST_out.txt) and the TPM-ranked transcript table (*.ig-transcripts.sortedbyTPM.txt) to identify the dominant (highest-TPM) Ig transcript, its top germline IGHV gene hit, and percent identity over the entire IGHV gene segment. Call SHM status as mCLL when percent identity < 98% and uCLL when ≥ 98% — using the full IGHV segment rather than a partial segment is what distinguishes CRIS from tools that infer the B-cell receptor from only a portion of the gene. Compile per-sample dominant IGHV gene, percent identity, and mCLL/uCLL calls, then compare to clinical Sanger classification; on GSE66228 the expected result is 16/16 concordance with Pearson r = 0.95 for percent-mutation values, and on EGAD00001004046 100% agreement for both SHM status and dominant-clone IGHV gene.

## Related tools

- **CRIS** (Pipeline that reconstructs IGHV from RNA-seq and predicts hypermutation (mCLL/uCLL) status) — https://github.com/Rashedul/CRIS
- **trinity v2.1.1** (De novo assembly of Ig transcripts extracted from the input BAM)
- **igblast v1.14.0** (Aligns assembled Ig transcripts to IMGT germline V/D/J sequences to obtain top IGHV hit and percent identity)
- **blast v2.9.0** (Alignment of contigs to reference Ig sequences used to enumerate somatic hypermutations)
- **seqkit v0.12.0** (Sequence handling for TPM-ranked Ig-transcript FASTA/tables)
- **salmon v0.8.1** (Quantifies Ig transcripts to produce TPM rankings used to select the dominant clone)
- **sambamba v0.7.0** (Coordinate-sorts and validates input BAMs)
- **picard v2.20.3** (BAM processing dependency of the CRIS pipeline)
- **jellyfish v2.2.10** (k-mer counting dependency of the CRIS pipeline)

## Examples

```
bash CRIS.sh -inbam SRR1814049_test.bam
```

## Evaluation signals

- Test bam SRR1814049_test.bam must reproduce IGHV3-74 with percent identity 94.6 (a mutated/mCLL call below the 98% cutoff)
- All four expected output files exist per sample: *.IgBLAST_out.txt, *.ig-transcripts.sortedbyTPM.txt, *.ig-transcripts.sortedbyTPM.fasta
- On GSE66228, CRIS mCLL/uCLL calls match clinical Sanger classification 16/16 with Pearson r = 0.95 for percent-mutation values
- On EGAD00001004046, dominant-clone IGHV gene and SHM status agree with clinical PCR-Sanger calls in 100% of samples
- Input BAMs verified as hg38-aligned, coordinate-sorted and indexed before running

## Limitations

- Requires minimum 16GB RAM and 4 threads; 25M reads minimum per sample
- Only applicable to BAMs aligned to hg38 (GRCh38) with BWA; other genome builds are not supported
- Validated in the context of Chronic Lymphocytic Leukemia; use for other B-cell malignancies is untested in this work

## Evidence

- [readme] CRIS has been validated against clinical PCR-Sanger based hypermutation classification in the context of Chronic Lymphocytic Leukemia.: "CRIS has been validated against clinical PCR-Sanger based hypermutation classification in the context of Chronic Lymphocytic Leukemia."
- [methods] SHM status call thresholds: [evidence span withheld — anchor status: not_found]
- [readme] Test run expected output: "# expected mutational status for test run 

- IGHV gene: IGHV3-74
- Percent identity: 94.6"
- [intro] Input requirements: [evidence span withheld — anchor status: not_found]
- [abstract] Concordance finding: "the SHM status identified by CRIS using the entire IGHV gene segment is highly concordant with clinical classification in three independent chronic lymphocytic leukemia patient cohorts"
