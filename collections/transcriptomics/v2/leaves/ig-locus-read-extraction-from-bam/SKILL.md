---
name: ig-locus-read-extraction-from-bam
description: Use when starting from a BWA-aligned, coordinate-sorted, indexed hg38
  RNA-seq BAM (e.g. ribodepleted or polyA-selected, ≥25M reads) and you need Ig-locus
  reads extracted and assembled into IGHV-D-J transcript contigs before SHM/IGHV mutation-status
  analysis.
when_to_use_negative:
- Input is not a BAM aligned to hg38 (GRCh38) with BWA, or is not coordinate-sorted
  and indexed — CRIS explicitly requires this pre-processing
- The goal is BCR repertoire inference from only a portion of the IGHV gene segment
  rather than de novo reconstruction of full IGHV-D-J transcripts
- RNA-seq data has fewer than 25M reads per sample, below CRIS's stated minimum
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3168
tools:
- name: CRIS
  role: Pipeline that orchestrates Ig-locus read extraction, FASTQ conversion, and
    Trinity de novo assembly of Ig transcripts
  repo: https://github.com/Rashedul/CRIS
- name: sambamba
  role: Extracts RNA-seq reads aligned to the putative Ig gene loci from the coordinate-sorted
    BAM (v0.7.0)
- name: picard
  role: SamToFastq converts the extracted Ig-locus alignments to paired-end FASTQ
    for assembly (v2.20.3)
- name: trinity
  role: De novo assembles the extracted FASTQ reads into Ig transcript contigs (v2.1.1),
    default 4 threads / 16G memory
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:23996e138b0676ce15b9846779a12061d434bfcc85169cd49af9e70eb9699ffb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Rashedul/CRIS
  tools:
  - CRIS
  - picard
  - sambamba
  - trinity
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ig-locus-read-extraction-from-bam

## Summary

Extracts RNA-seq read pairs aligned to putative immunoglobulin gene loci from a coordinate-sorted, indexed hg38 BAM using sambamba, converts them to paired-end FASTQ with Picard SamToFastq, and de novo assembles Ig transcript contigs with Trinity (v2.1.1) as the first stage of the CRIS IGHV-D-J reconstruction pipeline.

## When to use

Use when starting from a BWA-aligned, coordinate-sorted, indexed hg38 RNA-seq BAM (e.g. ribodepleted or polyA-selected, ≥25M reads) and you need Ig-locus reads extracted and assembled into IGHV-D-J transcript contigs before SHM/IGHV mutation-status analysis.

## When NOT to use

- Input is not a BAM aligned to hg38 (GRCh38) with BWA, or is not coordinate-sorted and indexed — CRIS explicitly requires this pre-processing
- The goal is BCR repertoire inference from only a portion of the IGHV gene segment rather than de novo reconstruction of full IGHV-D-J transcripts
- RNA-seq data has fewer than 25M reads per sample, below CRIS's stated minimum

## Inputs

- Coordinate-sorted, indexed BAM aligned to hg38 (GRCh38) using BWA (e.g. SRR1814049_test.bam)
- Putative immunoglobulin gene loci coordinates used by CRIS for sambamba extraction
- Minimum 16GB RAM and 4 threads available

## Outputs

- Paired-end FASTQ files of reads extracted from the putative Ig loci (via Picard SamToFastq)
- De novo Trinity-assembled Ig transcript contigs (subsequently quantified by TPM in ig-transcripts.sortedbyTPM.txt/.fasta)
- IGHV-D-J contigs suitable for downstream SHM enumeration and IGHV gene assignment (e.g. IGHV3-74 at 94.6 percent identity for the test BAM)

## How to apply

First verify the input BAM was aligned to hg38 (GRCh38) with BWA and is coordinate-sorted and indexed, as CRIS requires. Use sambamba (v0.7.0) to pull out the RNA-seq read pairs aligned to the putative immunoglobulin gene loci from the sorted BAM. Convert the extracted alignments to paired-end FASTQ with Picard SamToFastq (picard v2.20.3). De novo assemble the extracted FASTQ reads into Ig transcript contigs with Trinity (v2.1.1) under CRIS's default resource settings (threads=4, memory=16G; 16GB RAM and 4 threads are the stated minimums). Rationale: de novo assembly over the full IGHV gene segment is what distinguishes CRIS from tools that infer the B-cell receptor repertoire from only a portion of the IGHV segment, and the resulting contigs are what downstream steps (salmon TPM quantification, IgBLAST/BLAST alignment, SHM enumeration) consume.

## Related tools

- **CRIS** (Pipeline that orchestrates Ig-locus read extraction, FASTQ conversion, and Trinity de novo assembly of Ig transcripts) — https://github.com/Rashedul/CRIS
- **sambamba** (Extracts RNA-seq reads aligned to the putative Ig gene loci from the coordinate-sorted BAM (v0.7.0))
- **picard** (SamToFastq converts the extracted Ig-locus alignments to paired-end FASTQ for assembly (v2.20.3))
- **trinity** (De novo assembles the extracted FASTQ reads into Ig transcript contigs (v2.1.1), default 4 threads / 16G memory)

## Examples

```
bash CRIS.sh -inbam SRR1814049_test.bam -outdir /fullPath/ -threads 4 -memory 16G
```

## Evaluation signals

- The input BAM passes pre-checks: BWA-aligned to hg38, coordinate-sorted, and indexed
- Trinity assembly yields IGHV-D-J contigs rather than fragmented or empty output
- Test run on SRR1814049_test.bam recovers expected IGHV gene IGHV3-74 with percent identity 94.6
- Extracted read pairs round-trip correctly from BAM to paired-end FASTQ via Picard SamToFastq
- Resource settings match CRIS defaults (threads=4, memory=16G) and the machine meets the 16GB RAM / 4 thread minimum

## Limitations

- Requires BAM aligned specifically to hg38 (GRCh38) with BWA, coordinate-sorted and indexed; other builds or aligners are outside the validated workflow
- Minimum 25M sequence reads per sample; both ribodepleted and polyA-selected RNA-seq are supported but very low-input libraries may fail
- Minimum 16GB RAM and 4 threads required; assembly memory capped by the memory parameter (default 16G)

## Evidence

- [abstract] CRIS extracts RNA-seq reads aligned to Ig gene loci: "CRIS extracts RNA-seq reads aligned to Ig gene loci"
- [methods] Trinity de novo assembly of extracted reads: [evidence span withheld — anchor status: not_found]
- [intro] BAM preconditions for CRIS: [evidence span withheld — anchor status: not_found]
- [readme] README requirements block: "A minimum of 16GB RAM and 4 threads are required to run CRIS"
- [readme] Read-depth applicability: "Both ribodepleted and polyA selected RNA-seq datasets are appropriate with a minimum of 25M sequence reads per sample."
