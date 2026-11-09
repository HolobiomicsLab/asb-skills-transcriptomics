---
name: paired-end-fastq-generation-from-alignments
description: Use when an analysis needs FASTQ reads recovered from a subset of alignments
  in a coordinate-sorted, indexed BAM (e.g. reads mapped to putative Ig loci) so the
  reads can be fed to a de novo transcriptome assembler such as Trinity.
when_to_use_negative:
- The raw input is already FASTQ (e.g. from an unaligned RNA-seq run) — no BAM-to-FASTQ
  conversion is needed; proceed directly to assembly.
- You need only alignment-level information (positions, variants) from the BAM and
  do not intend to run a de novo assembler — reading the BAM directly is simpler than
  round-tripping to FASTQ.
- The BAM is not aligned to hg38 (GRCh38) with BWA, is not coordinate-sorted, or is
  not indexed — CRIS prerequisites are unmet and upstream re-alignment/sorting is
  required first.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: picard v2.20.3
  role: SamToFastq conversion of the extracted Ig-locus alignments into paired-end
    FASTQ reads
  repo: https://github.com/Rashedul/CRIS
- name: sambamba v0.7.0
  role: Extracts the RNA-seq reads aligned to the putative immunoglobulin gene loci
    from the sorted BAM prior to FASTQ conversion; also used for coordinate sorting
  repo: https://github.com/Rashedul/CRIS
- name: trinity v2.1.1
  role: 'Downstream consumer: de novo assembles the extracted paired-end FASTQ reads
    into Ig transcript contigs'
  repo: https://github.com/Rashedul/CRIS
- name: BWA
  role: Upstream aligner that produced the input BAM against hg38 (GRCh38)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:212d69c47a193e66fe651814fc2cb92f5b756af29f9796f4303ecc675b35eeac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Rashedul/CRIS
  tools:
  - BWA
  - CRIS
  - picard
  - picard v2.20.3
  - sambamba
  - sambamba v0.7.0
  - trinity
  - trinity v2.1.1
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# paired-end-fastq-generation-from-alignments

## Summary

Converts RNA-seq alignments extracted from immunoglobulin gene loci in a coordinate-sorted, indexed hg38 BAM into paired-end FASTQ reads with Picard SamToFastq, producing the input required for Trinity de novo assembly of Ig transcripts in the CRIS pipeline.

## When to use

Use when an analysis needs FASTQ reads recovered from a subset of alignments in a coordinate-sorted, indexed BAM (e.g. reads mapped to putative Ig loci) so the reads can be fed to a de novo transcriptome assembler such as Trinity.

## When NOT to use

- The raw input is already FASTQ (e.g. from an unaligned RNA-seq run) — no BAM-to-FASTQ conversion is needed; proceed directly to assembly.
- You need only alignment-level information (positions, variants) from the BAM and do not intend to run a de novo assembler — reading the BAM directly is simpler than round-tripping to FASTQ.
- The BAM is not aligned to hg38 (GRCh38) with BWA, is not coordinate-sorted, or is not indexed — CRIS prerequisites are unmet and upstream re-alignment/sorting is required first.

## Inputs

- Coordinate-sorted, indexed BAM aligned to hg38 (GRCh38) with BWA (e.g. SRR1814049_test.bam)
- Subset BAM of alignments to putative immunoglobulin gene loci produced by sambamba (v0.7.0) extraction

## Outputs

- Paired-end FASTQ files corresponding to the extracted Ig-locus alignments, suitable as Trinity v2.1.1 input

## How to apply

First ensure the input BAM is aligned to hg38 (GRCh38) with BWA, coordinate-sorted and indexed, as CRIS requires. Use sambamba (v0.7.0) to extract the RNA-seq reads aligned to the putative immunoglobulin gene loci from the sorted BAM. Then run Picard SamToFastq (picard v2.20.3) on the extracted alignments to convert them into paired-end FASTQ files, preserving read-pair correspondence so Trinity (v2.1.1) can de novo assemble the Ig transcript contigs. The FASTQ output is intermediate: correctness is judged downstream by whether Trinity recovers IGHV-D-J contigs (e.g. IGHV3-74 at 94.6 percent identity for the SRR1814049_test.bam test run).

## Related tools

- **picard v2.20.3** (SamToFastq conversion of the extracted Ig-locus alignments into paired-end FASTQ reads) — https://github.com/Rashedul/CRIS
- **sambamba v0.7.0** (Extracts the RNA-seq reads aligned to the putative immunoglobulin gene loci from the sorted BAM prior to FASTQ conversion; also used for coordinate sorting) — https://github.com/Rashedul/CRIS
- **trinity v2.1.1** (Downstream consumer: de novo assembles the extracted paired-end FASTQ reads into Ig transcript contigs) — https://github.com/Rashedul/CRIS
- **BWA** (Upstream aligner that produced the input BAM against hg38 (GRCh38))

## Examples

```
bash CRIS_docker.sh SRR1814049_test.bam threads=4 memory=16G
```

## Evaluation signals

- The conversion output must be paired-end FASTQ with mate information intact so Trinity de novo assembly can run on it with the CRIS default resource settings (threads=4, memory=16G).
- Assembly of the FASTQ output must produce IGHV-D-J contigs that can be aligned to reference Ig sequences.
- For the SRR1814049_test.bam test run, the downstream pipeline should recover IGHV gene IGHV3-74 at 94.6 percent identity.
- The extracted alignments must derive from a BAM aligned to the hg38 genome build, coordinate-sorted and indexed, matching CRIS requirements.

## Limitations

- The input BAM must be aligned to hg38 (GRCh38) using BWA and be coordinate-sorted and indexed; BAMs built against other references or unsorted BAMs are not supported by CRIS.
- CRIS requires at least 16GB RAM and 4 threads; default max assembly memory is 16G, which bounds the scale of the assembly step fed by these FASTQs.
- This step only recovers reads present as alignments in the input BAM; reads from the Ig loci that failed alignment are not recoverable at this stage.

## Evidence

- [methods] Convert the extracted alignments to paired-end FASTQ reads with Picard SamToFastq (picard v2.20.3).: [evidence span withheld — anchor status: not_found]
- [methods] Use sambamba (v0.7.0) to extract RNA-seq reads aligned to the putative immunoglobulin gene loci from the sorted BAM.: [evidence span withheld — anchor status: not_found]
- [readme] bam file must be aligned to hg38 (GRCh38) genome build using BWA, coordinate-sorted and indexed: "bam file must be aligned to hg38 (GRCh38) genome build using [BWA](http://bio-bwa.sourceforge.net/), coordinate-sorted and indexed"
- [methods] De novo assemble the extracted FASTQ reads into Ig transcript contigs using Trinity (v2.1.1) with the CRIS default resource settings (threads=4, memory=16G).: [evidence span withheld — anchor status: not_found]
