---
name: immunoglobulin-transcript-de-novo-assembly
description: Use when CLL RNA-seq BAMs aligned to hg38 (GRCh38) must yield the dominant
  IGHV gene, its percent identity to the germline, and mCLL/uCLL status, and partial-IGHV
  tools (e.g. V'DJer) are insufficient.
when_to_use_negative:
- The BAM is not aligned to hg38 or is not coordinate-sorted/indexed — CRIS requires
  this exact input state
- The goal is only a partial-IGHV repertoire sketch rather than full-length IGHV reconstruction
  and SHM classification
- Samples have far fewer than 25M reads, insufficient for de novo assembly of Ig transcripts
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2830
tools:
- name: CRIS
  role: Pipeline that extracts Ig-locus reads, assembles Ig transcripts, and classifies
    SHM status
  repo: https://github.com/Rashedul/CRIS
- name: trinity
  role: De novo assembly of Ig transcript contigs from extracted reads (v2.1.1)
- name: igblast
  role: Aligns assembled contigs to germline Ig sequences to identify top IGHV hit
    and percent identity (v1.14.0)
- name: blast
  role: Contig-to-reference Ig sequence alignment (v2.9.0)
- name: salmon
  role: Quantifies Ig transcript expression (TPM) used to sort transcripts and select
    the dominant clone (v0.8.1)
- name: sambamba
  role: Verifies/ensures BAMs are coordinate-sorted and indexed (v0.7.0)
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_008
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:df51768415b50f2dd5fabbd4c069286b9d45d3ec739fad1f336035934f4c3ff4
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
  - sambamba
  - seqkit
  - trinity
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# Immunoglobulin transcript de novo assembly from RNA-seq

## Summary

A CRIS-pipeline skill that extracts RNA-seq reads aligned to Ig gene loci from a hg38-aligned BAM, assembles Ig transcripts de novo with Trinity, and aligns contigs to reference Ig sequences with BLAST/IgBLAST to reconstruct the full IGHV region and classify somatic hypermutation status.

## When to use

Use when CLL RNA-seq BAMs aligned to hg38 (GRCh38) must yield the dominant IGHV gene, its percent identity to the germline, and mCLL/uCLL status, and partial-IGHV tools (e.g. V'DJer) are insufficient.

## When NOT to use

- The BAM is not aligned to hg38 or is not coordinate-sorted/indexed — CRIS requires this exact input state
- The goal is only a partial-IGHV repertoire sketch rather than full-length IGHV reconstruction and SHM classification
- Samples have far fewer than 25M reads, insufficient for de novo assembly of Ig transcripts

## Inputs

- Coordinate-sorted, indexed RNA-seq BAM aligned to hg38 (GRCh38) using BWA (e.g. CLL sample BAMs from GSE66228, EGAD00001004046, or phs000435.v3)
- Reference Ig germline sequences for BLAST/IgBLAST alignment
- Minimum 25M sequence reads per sample (ribodepleted or polyA-selected RNA-seq)

## Outputs

- <sample>.IgBLAST_out.txt — percent identity and alignment between Ig-transcript and top germline IGHV gene hits
- <sample>.ig-transcripts.sortedbyTPM.txt — Ig-transcript sequence IDs sorted by expression (TPM)
- <sample>.ig-transcripts.sortedbyTPM.fasta — Ig-transcript FASTA sequences sorted by TPM
- Per-sample dominant IGHV gene, percent identity, and mCLL/uCLL SHM classification

## How to apply

Start from a coordinate-sorted, indexed BAM aligned to hg38 with BWA (verify/sort with sambamba v0.7.0). Run CRIS (bash CRIS.sh -inbam <sample>.bam -outdir <dir> -threads 4 -memory 16G, or via the cris:v1 Docker image) so it extracts reads at Ig gene loci and assembles Ig transcripts de novo with Trinity v2.1.1. Align the resulting contigs to reference Ig sequences using blast v2.9.0 and igblast v1.14.0 to identify the top germline IGHV gene hit and its percent identity for the dominant (highest-TPM) transcript. Collect the per-sample outputs *.IgBLAST_out.txt, *.ig-transcripts.sortedbyTPM.txt and *.ig-transcripts.sortedbyTPM.fasta; classify SHM status as mutated (mCLL) vs unmutated (uCLL) from the percent identity to the top germline IGHV hit, using the full-length reconstruction — CRIS uses the entire IGHV gene segment, which matters near the 98% clinical cutoff because truncated assemblies can shift identity (e.g. 295/296 bp at 91.2% vs 226 bp at 89.4% for IGHV3-74 in sample US-1422368).

## Related tools

- **CRIS** (Pipeline that extracts Ig-locus reads, assembles Ig transcripts, and classifies SHM status) — https://github.com/Rashedul/CRIS
- **trinity** (De novo assembly of Ig transcript contigs from extracted reads (v2.1.1))
- **igblast** (Aligns assembled contigs to germline Ig sequences to identify top IGHV hit and percent identity (v1.14.0))
- **blast** (Contig-to-reference Ig sequence alignment (v2.9.0))
- **salmon** (Quantifies Ig transcript expression (TPM) used to sort transcripts and select the dominant clone (v0.8.1))
- **sambamba** (Verifies/ensures BAMs are coordinate-sorted and indexed (v0.7.0))

## Examples

```
bash CRIS.sh -inbam SRR1814049_test.bam -outdir /fullPath/ -threads 4 -memory 16G
```

## Evaluation signals

- Test run on the supplied SRR1814049_test.bam must return IGHV gene IGHV3-74 with percent identity 94.6
- All three expected output files exist per sample: *.IgBLAST_out.txt, *.ig-transcripts.sortedbyTPM.txt, *.ig-transcripts.sortedbyTPM.fasta
- The dominant (highest-TPM) Ig transcript contig aligns to a single top germline IGHV gene with a computed percent identity
- CRIS mCLL/uCLL calls match clinical Sanger classification (benchmark: 16/16 in GSE66228; 50/51 with 98.3% accuracy, 100% sensitivity, 97.3% specificity in phs000435.v3)
- Reconstruction length approaches full IGHV length (e.g. 295/296 bp), with longer reconstructions than partial tools like V'DJer (226 bp)

## Limitations

- Requires a minimum of 25M reads per sample and at least 16GB RAM / 4 threads
- Input BAM must be aligned to hg38 (GRCh38) with BWA, coordinate-sorted and indexed; other builds or sort orders are unsupported
- Full-length reconstruction changes percent identity relative to partial-assembly tools (e.g. 91.2% vs 89.4% for IGHV3-74 in US-1422368), so results are not directly comparable to truncated-IGHV tools near classification cutoffs
- Single discordant cases occur (e.g. sample DFCI-5121: Sanger mCLL vs CRIS uCLL in phs000435.v3)

## Evidence

- [abstract] CRIS extracts RNA-seq reads aligned to Ig gene loci, performs assembly of Ig transcripts and aligns the resulting contigs to reference Ig sequences: "CRIS extracts RNA-seq reads aligned to Ig gene loci, performs assembly of Ig transcripts and aligns the resulting contigs to reference Ig sequences"
- [intro] bam file must be aligned to hg38 (GRCh38) genome build using BWA, coordinate-sorted and indexed: [evidence span withheld — anchor status: not_found]
- [readme] to confirm install please run CRIS using the supplied test bam file (SRR1814049_test.bam): "to confirm install please run CRIS using the supplied test bam file (SRR1814049_test.bam)"
- [readme] Contains the percent identity and alignment between Ig-transcript and top germline IGHV gene hits.: "Contains the percent identity and alignment between Ig-transcript and top germline IGHV gene hits."
- [readme] Both ribodepleted and polyA selected RNA-seq datasets are appropriate with a minimum of 25M sequence reads per sample: "Both ribodepleted and polyA selected RNA-seq datasets are appropriate with a minimum of 25M sequence reads per sample"
- [abstract] the SHM status identified by CRIS using the entire IGHV gene segment is highly concordant with clinical classification in three independent chronic lymphocytic leukemia patient cohorts: "We show that the SHM status identified by CRIS using the entire IGHV gene segment is highly concordant with clinical classification in three independent chronic lymphocytic leukemia patient cohorts"
