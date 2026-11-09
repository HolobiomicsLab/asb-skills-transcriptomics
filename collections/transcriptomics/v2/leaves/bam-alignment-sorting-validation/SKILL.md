---
name: bam-alignment-sorting-validation
description: Use when a BAM file is about to be fed into CRIS (or an analogous RNA-seq
  Ig reconstruction pipeline) and you must verify or produce hg38/GRCh38 alignment,
  coordinate sorting, and an index before running.
when_to_use_negative:
- The alignment is to a genome build other than hg38 (GRCh38) — CRIS requires the
  BAM to be BWA-aligned to hg38, so re-alignment is needed first
- Input is raw FASTQ or an already-assembled transcriptome/contig set rather than
  an aligned BAM — this skill only validates/produces the BAM stage of the CRIS workflow
- The goal is SHM or IGHV-status interpretation from an existing CRIS run; this skill
  covers only upstream BAM preparation, not mutation enumeration
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: BWA
  role: Aligns the RNA-seq reads to the hg38 (GRCh38) genome build so the resulting
    BAM meets CRIS's input requirement
  repo: http://bio-bwa.sourceforge.net/
- name: sambamba (v0.7.0)
  role: Coordinate-sorts the BWA-aligned BAM (and subsequently extracts reads from
    the putative Ig loci) as required before CRIS runs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:c9e4d0f7b373868cb05447abcfb00c2eb95e139049597731faba788f536a9f8b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA
  - CRIS
  - picard
  - sambamba
  - sambamba (v0.7.0)
  - trinity
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# bam-alignment-sorting-validation

## Summary

Validates and prepares the CRIS-required input BAM: an RNA-seq alignment to the hg38 (GRCh38) build produced with BWA, coordinate-sorted with sambamba, and indexed, so downstream Ig-locus read extraction and Trinity assembly succeed.

## When to use

Use when a BAM file is about to be fed into CRIS (or an analogous RNA-seq Ig reconstruction pipeline) and you must verify or produce hg38/GRCh38 alignment, coordinate sorting, and an index before running.

## When NOT to use

- The alignment is to a genome build other than hg38 (GRCh38) — CRIS requires the BAM to be BWA-aligned to hg38, so re-alignment is needed first
- Input is raw FASTQ or an already-assembled transcriptome/contig set rather than an aligned BAM — this skill only validates/produces the BAM stage of the CRIS workflow
- The goal is SHM or IGHV-status interpretation from an existing CRIS run; this skill covers only upstream BAM preparation, not mutation enumeration

## Inputs

- Input RNA-seq BAM file aligned to hg38 (GRCh38) with BWA (e.g. SRR1814049_test.bam)
- BAM index for the coordinate-sorted alignment
- Input RNA-seq library type: ribodepleted or polyA-selected, >=25M reads per sample

## Outputs

- Validated, coordinate-sorted, indexed BAM ready for CRIS Ig-locus read extraction
- Confirmation of pipeline readiness via the CRIS test run output (IGHV gene: IGHV3-74, percent identity: 94.6)

## How to apply

First confirm the input BAM was aligned to the hg38 (GRCh38) genome build using BWA, since CRIS extracts reads from putative immunoglobulin gene loci by genomic coordinates. Then ensure the BAM is coordinate-sorted — the article/README specify sorting with sambamba (v0.7.0) sort — and indexed. Check sample suitability: both ribodepleted and polyA-selected RNA-seq are acceptable, but the sample should have a minimum of 25M sequence reads. Confirm compute resources (minimum 16GB RAM and 4 threads; CRIS defaults to threads=4, memory=16G). Finally, validate the setup end-to-end by running CRIS on the supplied test BAM, SRR1814049_test.bam, which should reconstruct IGHV3-74 at 94.6% identity.

## Related tools

- **BWA** (Aligns the RNA-seq reads to the hg38 (GRCh38) genome build so the resulting BAM meets CRIS's input requirement) — http://bio-bwa.sourceforge.net/
- **sambamba (v0.7.0)** (Coordinate-sorts the BWA-aligned BAM (and subsequently extracts reads from the putative Ig loci) as required before CRIS runs)

## Examples

```
sudo docker run --name cris_analysis -v /fullPath/SRR1814049_test.bam:/cris/SRR1814049_test.bam -t cris:v1 bash CRIS_docker.sh SRR1814049_test.bam threads=4 memory=16G
```

## Evaluation signals

- The BAM header/reference checks confirm alignment is to hg38 (GRCh38), not an older or alternative build
- The BAM is coordinate-sorted and an index file exists; the article specifies SAMBAMBA sort for this step
- A CRIS test run with the supplied SRR1814049_test.bam returns the expected result: IGHV gene IGHV3-74 with percent identity 94.6
- Sample depth is at least 25M sequence reads, and ≥16GB RAM / 4 threads were available (CRIS default threads=4, memory=16G)

## Limitations

- CRIS only accepts BAMs aligned to the hg38 (GRCh38) build with BWA; alignments to other builds or aligners are out of scope
- A minimum of 25M sequence reads per sample (ribo-depleted or polyA-selected) is stated as the input floor; fewer reads may compromise IGHV reconstruction
- At least 16GB RAM and 4 threads are required; the supplied test BAM (SRR1814049_test.bam) is the only documented validation case (IGHV3-74 at 94.6% identity)

## Evidence

- [readme] bam file must be aligned to hg38 (GRCh38) genome build using [BWA](http://bio-bwa.sourceforge.net/), coordinate-sorted and indexed: "bam file must be aligned to hg38 (GRCh38) genome build using [BWA](http://bio-bwa.sourceforge.net/), coordinate-sorted and indexed"
- [intro] coordinate-sorted using SAMBAMBA sort: "coordinate-sorted using SAMBAMBA sort"
- [readme] Both ribodepleted and polyA selected RNA-seq datasets are appropriate with a minimum of 25M sequence reads per sample.: "Both ribodepleted and polyA selected RNA-seq datasets are appropriate with a minimum of 25M sequence reads per sample."
- [readme] to confirm install please run CRIS using the supplied test bam file (SRR1814049_test.bam): "to confirm install please run CRIS using the supplied test bam file (SRR1814049_test.bam) that has been aligned to hg38 (GRCh38) build"
