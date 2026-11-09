---
name: igblast-germline-vdj-alignment
description: Use when you have RNA-seq-derived Ig transcript FASTAs (TPM-ranked) or
  contigs from a BCR/CLL sample and need to assign a germline IGHV gene, compute percent
  identity, and call mCLL/uCLL SHM status.
when_to_use_negative:
- The input is raw RNA-seq FASTQ reads or an unprocessed whole-transcriptome BAM —
  Ig transcripts must first be extracted from Ig gene loci and de novo assembled (CRIS
  steps 1–2) before IgBLAST alignment is meaningful
- The BAM is not aligned to hg38 (GRCh38) with BWA, coordinate-sorted and indexed
  — CRIS will not accept it and the germline-alignment step cannot be validated
- The goal is to infer the B-cell receptor repertoire from only a portion of the IGHV
  gene segment — CRIS deliberately improves on such tools by reconstructing the entire
  IGHV segment via de novo assembly
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3676
tools:
- name: igblast
  role: Aligns each assembled Ig transcript to the IMGT germline V/D/J database and
    returns the top germline IGHV hit and percent identity (v1.14.0 in CRIS bash mode)
  repo: https://github.com/Rashedul/CRIS
- name: blast
  role: Provides the alignment engine (v2.9.0) underlying CRIS's IgBLAST-based mutation
    enumeration against germline IGHV
  repo: https://github.com/Rashedul/CRIS
- name: seqkit
  role: Sequence handling (v0.12.0) of the TPM-ranked Ig-transcript FASTA fed into
    the germline alignment and clonotype clustering
  repo: https://github.com/Rashedul/CRIS
- name: salmon
  role: Quantifies TPM expression (v0.8.1) used to rank Ig transcripts and prioritize
    the dominant clone's alignment for the IGHV call
  repo: https://github.com/Rashedul/CRIS
- name: trinity
  role: De novo assembles (v2.1.1) the Ig transcripts from extracted reads, producing
    the contigs that this skill aligns to germline
  repo: https://github.com/Rashedul/CRIS
provenance:
  source_task_ids:
  - task_007
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:9dc69ce1e725586f9ada31f6d1e3651b848dae79372d3b2ea6b41624440c1447
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Rashedul/CRIS
  tools:
  - CRIS
  - Docker
  - blast
  - igblast
  - jellyfish
  - picard
  - salmon
  - seqkit
  - trinity
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3676
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# igblast-germline-vdj-alignment

## Summary

Aligns de novo-assembled Ig transcripts (or contigs) against the IMGT germline V, D and J reference database with IgBLAST to recover the top germline IGHV hit, its percent identity, somatic hypermutations, and clonotype groupings — the mutation-calling core of the CRIS CLL IGHV classification pipeline.

## When to use

Use when you have RNA-seq-derived Ig transcript FASTAs (TPM-ranked) or contigs from a BCR/CLL sample and need to assign a germline IGHV gene, compute percent identity, and call mCLL/uCLL SHM status.

## When NOT to use

- The input is raw RNA-seq FASTQ reads or an unprocessed whole-transcriptome BAM — Ig transcripts must first be extracted from Ig gene loci and de novo assembled (CRIS steps 1–2) before IgBLAST alignment is meaningful
- The BAM is not aligned to hg38 (GRCh38) with BWA, coordinate-sorted and indexed — CRIS will not accept it and the germline-alignment step cannot be validated
- The goal is to infer the B-cell receptor repertoire from only a portion of the IGHV gene segment — CRIS deliberately improves on such tools by reconstructing the entire IGHV segment via de novo assembly

## Inputs

- TPM-ranked Ig-transcript FASTA (e.g. SRR1814049_test.bam.ig-transcripts.sortedbyTPM.fasta) from CRIS step 2
- Ig-transcript sequence IDs sorted by expression (SRR1814049_test.bam.ig-transcripts.sortedbyTPM.txt)
- IMGT germline V, D and J reference database for IgBLAST
- Input BAM aligned to hg38 (GRCh38) with BWA, coordinate-sorted and indexed

## Outputs

- IgBLAST output file (e.g. SRR1814049_test.bam.IgBLAST_out.txt) with percent identity and alignment of each Ig-transcript to its top germline IGHV gene hit
- Per-sample structured record: dominant IGHV gene, percent identity, and mCLL/uCLL SHM call
- Clonotype table assigning each Ig transcript to its V-D-J clonotype
- Cohort-level IGHV call table (e.g. EGAD00001004046_ighv_calls.tsv) with per-sample gene/identity/call and Sanger concordance

## How to apply

Load the TPM-ranked Ig-transcript FASTA and transcript-ID table produced by the transcript-assembly step of CRIS (trinity v2.1.1 assembly from reads extracted at Ig loci of a hg38/BWA BAM). Align each Ig transcript against the germline V/D/J reference sequences with IgBLAST (v1.14.0; BLAST v2.9.0-based alignment) to obtain the top germline IGHV gene hit and its percent identity, then enumerate somatic hypermutations by comparing each transcript alignment to that germline hit. Cluster transcripts sharing the same V-D-J rearrangement into clonotypes, prioritizing transcripts by TPM expression ranking (seqkit v0.12.0 for sequence handling; salmon v0.8.1 quantifies TPM). Call SHM status from percent identity to the top germline IGHV hit using the 98% cutoff: identity < 98% = mCLL (mutated), identity >= 98% = uCLL (unmutated), and record the dominant (highest-TPM) clone's IGHV gene, percent identity, and call. Parse the per-sample IgBLAST output file to extract these fields for downstream concordance comparison against clinical PCR-Sanger classification.

## Related tools

- **igblast** (Aligns each assembled Ig transcript to the IMGT germline V/D/J database and returns the top germline IGHV hit and percent identity (v1.14.0 in CRIS bash mode)) — https://github.com/Rashedul/CRIS
- **blast** (Provides the alignment engine (v2.9.0) underlying CRIS's IgBLAST-based mutation enumeration against germline IGHV) — https://github.com/Rashedul/CRIS
- **seqkit** (Sequence handling (v0.12.0) of the TPM-ranked Ig-transcript FASTA fed into the germline alignment and clonotype clustering) — https://github.com/Rashedul/CRIS
- **salmon** (Quantifies TPM expression (v0.8.1) used to rank Ig transcripts and prioritize the dominant clone's alignment for the IGHV call) — https://github.com/Rashedul/CRIS
- **trinity** (De novo assembles (v2.1.1) the Ig transcripts from extracted reads, producing the contigs that this skill aligns to germline) — https://github.com/Rashedul/CRIS

## Examples

```
bash CRIS.sh -inbam SRR1814049_test.bam
```

## Evaluation signals

- Test-run reproduction: running CRIS on SRR1814049_test.bam must yield IGHV gene IGHV3-74 with percent identity 94.6
- The 94.6% identity result must be classified as mCLL (mutated) under the < 98% identity cutoff
- On phs000435.v3 (51 samples), calls must match Sanger in 50/51 cases (98.3% accuracy, 100% sensitivity, 97.3% specificity), with DFCI-5121 (Sanger mCLL vs CRIS uCLL) as the single discordant case
- On EGAD00001004046, CRIS SHM status and dominant-clone IGHV gene must match clinical Sanger classification in 100% of cohort samples
- Output file SRR1814049_test.bam.IgBLAST_out.txt must exist and contain the percent identity and alignment between Ig-transcript and top germline IGHV gene hits

## Limitations

- Requires a minimum of 25M sequence reads per sample and at least 16GB RAM / 4 threads; both ribodepleted and polyA-selected RNA-seq are acceptable
- Input BAM must be aligned to the hg38 (GRCh38) genome build using BWA, coordinate-sorted and indexed, otherwise the pipeline is not supported
- One known discordance exists in phs000435.v3 (sample DFCI-5121, Sanger mCLL vs CRIS uCLL), so per-sample concordance is not guaranteed at the boundary of the 98% identity cutoff
- Validation context is chronic lymphocytic leukemia IGHV classification; no discussion-section caveats were available in the extracted article content

## Evidence

- [abstract] performs assembly of Ig transcripts and aligns the resulting contigs to reference Ig sequences: "CRIS extracts RNA-seq reads aligned to Ig gene loci, performs assembly of Ig transcripts and aligns the resulting contigs to reference Ig sequences"
- [readme] igblast (v1.14.0), salmon (v0.8.1): "picard (v2.20.3), trinity (v2.1.1), blast (v2.9.0), seqkit (v0.12.0), sambamba (v0.7.0), salmon (v0.8.1), igblast (v1.14.0), jellyfish (v2.2.10)"
- [readme] SRR1814049_test.bam.IgBLAST_out.txt: "SRR1814049_test.bam.IgBLAST_out.txt Contains the percent identity and alignment between Ig-transcript and top germline IGHV gene hits."
- [methods] mCLL (mutated, identity < 98%) or uCLL (unmutated, identity >= 98%): [evidence span withheld — anchor status: not_found]
- [methods] IGHV3-74 with percent identity 94.6: [evidence span withheld — anchor status: not_found]
