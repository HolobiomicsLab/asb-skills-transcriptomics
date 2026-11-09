---
name: rnaseq-alignment-grch38
description: Use when RNA-seq reads (e.g. CLL samples like GSE66228) must be converted
  into an hg38-aligned, coordinate-sorted, indexed BAM suitable for CRIS or comparable
  BCR-reconstruction tools.
when_to_use_negative:
- The input BAM is already aligned to a genome build other than hg38 (GRCh38) — CRIS
  requires hg38 alignment, so the sample must be realigned rather than reused.
- The goal is de novo IGHV assembly output itself (CRIS, V'DJer, TRUST, MiXCR run)
  rather than producing the sorted/indexed hg38 BAM that feeds those tools.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BWA
  role: Reference aligner named by the paper for producing the hg38-aligned BAM required
    by CRIS
  repo: http://bio-bwa.sourceforge.net/
- name: SAMBAmba
  role: Coordinate-sorts the aligned BAM (sambamba sort) so it is valid CRIS input
- name: STAR
  role: Spliced aligner used in the GSE66228 benchmark workflow to align the 16 CLL
    RNA-seq samples to GRCh38 and emit coordinate-sorted, indexed BAMs
- name: CRIS
  role: Downstream consumer of the sorted/indexed hg38 BAM; its test run validates
    that the alignment step was performed correctly
  repo: https://github.com/Rashedul/CRIS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbab021
    title: 'CRIS: complete reconstruction of immunoglobulin <i>V-D-J</i> sequences
      from RNA-seq data'
  source_content_hash: sha256:87c1f8a6829749c21106ac217ca746ecb033365f31f31aa2f7dd914ba0435d99
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA
  - CRIS
  - Docker
  - MiXCR
  - SAMBAmba
  - STAR
  - TRUST
  - V'DJer
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# rnaseq-alignment-grch38

## Summary

Align RNA-seq reads to the hg38 (GRCh38) genome build and produce a coordinate-sorted, indexed BAM — the mandatory input format for CRIS's IGHV reconstruction and for benchmarking against V'DJer, TRUST, and MiXCR.

## When to use

Use when RNA-seq reads (e.g. CLL samples like GSE66228) must be converted into an hg38-aligned, coordinate-sorted, indexed BAM suitable for CRIS or comparable BCR-reconstruction tools.

## When NOT to use

- The input BAM is already aligned to a genome build other than hg38 (GRCh38) — CRIS requires hg38 alignment, so the sample must be realigned rather than reused.
- The goal is de novo IGHV assembly output itself (CRIS, V'DJer, TRUST, MiXCR run) rather than producing the sorted/indexed hg38 BAM that feeds those tools.

## Inputs

- RNA-seq FASTQ reads (ribodepleted or polyA-selected, minimum 25M reads per sample)
- GRCh38 (hg38) reference genome build / alignment index

## Outputs

- BAM file aligned to hg38 (GRCh38), coordinate-sorted and indexed (with .bai index)
- CRIS IgBLAST output (e.g. SRR1814049_test.bam.IgBLAST_out.txt) when the test BAM is run to confirm the alignment is valid

## How to apply

Align the RNA-seq reads to the hg38 (GRCh38) reference; the CRIS paper specifies BWA, while the GSE66228 benchmark workflow used STAR at 16 threads, so either aligner is acceptable as long as the output is an hg38-aligned BAM. Sort the resulting BAM by coordinate (the paper used SAMBAMBA sort) and index it, since CRIS rejects BAMs that are not coordinate-sorted and indexed. Verify the setup before batch runs by executing CRIS with the supplied test file SRR1814049_test.bam, which must return IGHV gene IGHV3-74 with percent identity 94.6. Use a minimum of 25M sequence reads per sample; both ribodepleted and polyA-selected RNA-seq are appropriate. Allocate at least 16G memory and 4 threads for the downstream CRIS run (16 threads/16G was used in the four-tool benchmark).

## Related tools

- **BWA** (Reference aligner named by the paper for producing the hg38-aligned BAM required by CRIS) — http://bio-bwa.sourceforge.net/
- **SAMBAmba** (Coordinate-sorts the aligned BAM (sambamba sort) so it is valid CRIS input)
- **STAR** (Spliced aligner used in the GSE66228 benchmark workflow to align the 16 CLL RNA-seq samples to GRCh38 and emit coordinate-sorted, indexed BAMs)
- **CRIS** (Downstream consumer of the sorted/indexed hg38 BAM; its test run validates that the alignment step was performed correctly) — https://github.com/Rashedul/CRIS

## Examples

```
bash CRIS.sh -inbam SRR1814049_test.bam -threads 16 -memory 16G
```

## Evaluation signals

- BAM header reports SO:coordinate (coordinate-sorted) and a matching .bai index exists
- BAM contig names correspond to the hg38 (GRCh38) build, not hg19 or another build
- Running CRIS on the supplied SRR1814049_test.bam returns IGHV gene IGHV3-74 with percent identity 94.6
- CRIS test-run outputs (IgBLAST_out.txt, ig-transcripts sorted-by-TPM .txt and .fasta) are generated without errors
- Each sample has at least 25M reads, within the validated input range for CRIS

## Limitations

- Only hg38 (GRCh38) alignment is supported; BAMs aligned with other builds or unsorted/indexed BAMs are invalid CRIS input.
- A minimum of 25M sequence reads per sample and 16GB RAM / 4 threads are required; low-depth libraries may fail IGHV reconstruction.
- The README specifies BWA as the aligner, so deviations (e.g. STAR) are validated only by the benchmark workflow, not by the paper itself.

## Evidence

- [intro] bam file must be aligned to hg38 (GRCh38) genome build using BWA: [evidence span withheld — anchor status: not_found]
- [readme] Both ribodepleted and polyA selected RNA-seq datasets are appropriate with a minimum of 25M sequence reads per sample.: "Both ribodepleted and polyA selected RNA-seq datasets are appropriate with a minimum of 25M sequence reads per sample."
- [readme] coordinate-sorted using SAMBAMBA sort: "supplied test bam file (SRR1814049_test.bam) that has been aligned to hg38 (GRCh38) build and coordinate-sorted using SAMBAMBA sort"
- [methods] expected mutational status for test run: "- IGHV gene: IGHV3-74
- Percent identity: 94.6"
