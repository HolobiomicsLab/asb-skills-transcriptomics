---
name: immunoglobulin-heavy-chain-locus-subsetting
description: Use when starting from a bulk RNA-seq BAM aligned to hg38 (GRCh38) with
  BWA, coordinate-sorted and indexed, and the goal is to subset reads from putative
  Ig loci and de novo assemble IGHV-D-J transcript contigs prior to SHM analysis.
when_to_use_negative:
- The input BAM is not aligned to hg38 (GRCh38) with BWA, or is not coordinate-sorted
  and indexed — CRIS will not operate correctly on such files
- The data is targeted/amplicon B-cell receptor repertoire sequencing (Ig-seq) rather
  than bulk RNA-seq (ribodepleted or polyA selected, >=25M reads per sample) — de
  novo locus subsetting from a whole-transcriptome alignment is unnecessary
- You already have assembled Ig transcript contigs and only need downstream SHM classification
  or contig-to-reference Ig alignment steps
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: CRIS
  role: Pipeline that orchestrates Ig-loci read extraction and downstream assembly;
    run via CRIS.sh or the Docker CRIS_docker.sh wrapper
  repo: https://github.com/Rashedul/CRIS
- name: sambamba
  role: Extracts RNA-seq reads aligned to the putative immunoglobulin gene loci from
    the sorted, indexed BAM (v0.7.0)
- name: picard
  role: Converts the extracted Ig-locus alignments to paired-end FASTQ reads via SamToFastq
    (v2.20.3)
- name: trinity
  role: De novo assembles the extracted Ig-locus FASTQ reads into Ig transcript contigs
    (v2.1.1)
- name: BWA
  role: 'Required upstream aligner: the input BAM must be aligned to hg38 (GRCh38)
    with BWA before locus subsetting'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:92093bddddab11f194808f8fb7cfb08c8d98b47bec2805b50201ffb89646cac3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Rashedul/CRIS
  tools:
  - BWA
  - CRIS
  - picard
  - sambamba
  - trinity
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Immunoglobulin heavy-chain locus subsetting

## Summary

Extracts RNA-seq reads aligned to the putative immunoglobulin gene loci from a hg38-aligned, coordinate-sorted BAM using sambamba, converts them to paired-end FASTQ with Picard SamToFastq, and de novo assembles the subset into Ig transcripts with Trinity — the read-extraction and assembly front end of the CRIS IGHV-D-J reconstruction pipeline.

## When to use

Use when starting from a bulk RNA-seq BAM aligned to hg38 (GRCh38) with BWA, coordinate-sorted and indexed, and the goal is to subset reads from putative Ig loci and de novo assemble IGHV-D-J transcript contigs prior to SHM analysis.

## When NOT to use

- The input BAM is not aligned to hg38 (GRCh38) with BWA, or is not coordinate-sorted and indexed — CRIS will not operate correctly on such files
- The data is targeted/amplicon B-cell receptor repertoire sequencing (Ig-seq) rather than bulk RNA-seq (ribodepleted or polyA selected, >=25M reads per sample) — de novo locus subsetting from a whole-transcriptome alignment is unnecessary
- You already have assembled Ig transcript contigs and only need downstream SHM classification or contig-to-reference Ig alignment steps

## Inputs

- Coordinate-sorted, indexed RNA-seq BAM aligned to hg38 (GRCh38) with BWA (e.g. SRR1814049_test.bam)
- Putative immunoglobulin gene loci coordinates (as shipped with CRIS) used to subset alignments
- Trinity assembly resource settings: threads=4, memory=16G (defaults)

## Outputs

- Paired-end FASTQ files of reads extracted from the putative Ig loci (via Picard SamToFastq)
- De novo assembled Ig transcript contigs (Trinity FASTA) carrying IGHV-D-J sequences, passed onward for SHM enumeration

## How to apply

First verify the input BAM is aligned to the hg38 (GRCh38) genome build with BWA and is coordinate-sorted (e.g. via SAMBAMBA sort) and indexed, as CRIS requires. Use sambamba (v0.7.0) to extract the RNA-seq reads that align to the putative immunoglobulin gene loci from the sorted BAM. Convert the extracted alignments to paired-end FASTQ with Picard SamToFastq (picard v2.20.3). De novo assemble the extracted FASTQ reads into Ig transcript contigs with Trinity (v2.1.1), using CRIS's default resource settings of 4 threads and 16G memory (a minimum of 16GB RAM and 4 threads is required). Validate the assembly by confirming recovery of IGHV-D-J contigs that align to a germline IGHV gene (test case: IGHV3-74 at 94.6 percent identity for SRR1814049_test.bam).

## Related tools

- **CRIS** (Pipeline that orchestrates Ig-loci read extraction and downstream assembly; run via CRIS.sh or the Docker CRIS_docker.sh wrapper) — https://github.com/Rashedul/CRIS
- **sambamba** (Extracts RNA-seq reads aligned to the putative immunoglobulin gene loci from the sorted, indexed BAM (v0.7.0))
- **picard** (Converts the extracted Ig-locus alignments to paired-end FASTQ reads via SamToFastq (v2.20.3))
- **trinity** (De novo assembles the extracted Ig-locus FASTQ reads into Ig transcript contigs (v2.1.1))
- **BWA** (Required upstream aligner: the input BAM must be aligned to hg38 (GRCh38) with BWA before locus subsetting)

## Examples

```
bash CRIS.sh -inbam SRR1814049_test.bam -outdir /fullPath/ -threads 4 -memory 16G
```

## Evaluation signals

- Extraction produces paired-end FASTQ reads from the Ig loci; the FASTQ must be regenerable from the input BAM via Picard SamToFastq
- Trinity assembly yields contigs containing IGHV-D-J sequence (not just V-region fragments)
- Test-bam run on SRR1814049_test.bam reports expected IGHV gene IGHV3-74 with percent identity 94.6
- Input BAM passes preconditions: hg38 (GRCh38) alignment via BWA, coordinate-sorted and indexed
- Default resources honoured: 4 threads and 16G memory for the Trinity assembly step

## Limitations

- Requires at least 16GB RAM and 4 threads to run; Trinity assembly step defaults to 16G maximum memory
- Only bulk RNA-seq with a minimum of 25M reads per sample is appropriate (ribodepleted or polyA selected); shallower libraries may fail to recover Ig transcripts
- BAM must be aligned to hg38 (GRCh38) with BWA and be coordinate-sorted and indexed; other genome builds or aligners are outside the validated scope
- CRIS improves on tools that use only a portion of the IGHV gene segment by de novo assembly, but the extracted reads are limited to the putative Ig loci — reads outside those loci are not considered

## Evidence

- [abstract] CRIS extracts RNA-seq reads aligned to Ig gene loci: "CRIS extracts RNA-seq reads aligned to Ig gene loci"
- [readme] bam file must be aligned to hg38 (GRCh38) genome build using BWA, coordinate-sorted and indexed: [evidence span withheld — anchor status: not_found]
- [readme] Dependencies: picard (v2.20.3), trinity (v2.1.1), blast (v2.9.0), seqkit (v0.12.0), sambamba (v0.7.0): "picard (v2.20.3), trinity (v2.1.1), blast (v2.9.0), seqkit (v0.12.0), sambamba (v0.7.0)"
- [abstract] CRIS improves on existing tools that infer the B-cell receptor repertoire from RNA-seq data using a portion IGHV gene segment by de novo assembly: "CRIS improves on existing tools that infer the B-cell receptor repertoire from RNA-seq data using a portion IGHV gene segment by de novo assembly"
