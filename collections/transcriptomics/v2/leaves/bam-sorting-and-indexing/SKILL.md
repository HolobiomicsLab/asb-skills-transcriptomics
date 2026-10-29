---
name: bam-sorting-and-indexing
description: Reference for converting merged SAM alignments from Ribo-seq double-mapping
  (HISAT2 splice-aware followed by Bowtie2 mismatch-tolerant) into sorted, indexed
  BAM/BAI files ready for read counting and periodicity analysis.
when_to_use_negative:
- Alignments are still unmapped or multimapping reads have not been filtered (only
  uniquely mapped reads should be retained before counting)
- Input is raw FastQ or rRNA-contaminated reads that have not passed through Cutadapt
  trimming, Bowtie2 contaminant depletion, and the HISAT2/Bowtie2 genome alignment
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: samtools
  role: Sort, convert, and index the merged HISAT2/Bowtie2 alignments into BAM and
    BAI files
  repo: https://sourceforge.net/projects/samtools/files/samtools/
- name: RiboDoc
  role: Docker/Snakemake pipeline that orchestrates the sort-and-index step after
    genome alignment
  repo: https://github.com/equipeGST/RiboDoc
- name: HISAT2
  role: Produces the primary splice-aware SAM alignments that are merged and sorted
  repo: http://daehwankimlab.github.io/hisat2/
- name: Bowtie 2
  role: Remaps HISAT2-unaligned reads (≤1 mismatch); its SAM output feeds the merged,
    sorted BAM
  repo: https://bowtie-bio.sourceforge.net/bowtie2/index.shtml
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:64550a0f9d060cbb20e3eaaf240f66b9c1e628dec41614ed85a220821c64c94d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/equipeGST/RiboDoc
  tools:
  - Bowtie 2
  - HISAT2
  - RiboDoc
  - samtools
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# bam-sorting-and-indexing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sort and index merged ribosome-profiling alignments (HISAT2 + Bowtie2 SAM output) with samtools to produce coordinate-sorted BAM and BAI files in the BAM.25-35 results folder, enabling downstream counting (htseq-count) and P-site analysis (riboWaltz/TRiP) in the RiboDoc pipeline.

## When to use

Reference for converting merged SAM alignments from Ribo-seq double-mapping (HISAT2 splice-aware followed by Bowtie2 mismatch-tolerant) into sorted, indexed BAM/BAI files ready for read counting and periodicity analysis.

## When NOT to use

- Alignments are still unmapped or multimapping reads have not been filtered (only uniquely mapped reads should be retained before counting)
- Input is raw FastQ or rRNA-contaminated reads that have not passed through Cutadapt trimming, Bowtie2 contaminant depletion, and the HISAT2/Bowtie2 genome alignment

## Inputs

- Merged SAM files from HISAT2 and Bowtie2 alignment of rRNA-depleted, 25-35 nt RPF reads
- Per-sample alignment outputs from the RiboDoc double-mapping step

## Outputs

- Coordinate-sorted BAM files in the BAM.25-35 results folder
- Companion BAI index files (.bai) per sample

## How to apply

After the double-mapping step, where rRNA-depleted, 25-35 nt RPF reads are first aligned splice-aware with HISAT2 and unaligned reads are remapped with Bowtie2 (v2.4.2, accepting at most 1 mismatch), merge the two SAM outputs into a single alignment per sample and discard reads that remain unmapped. Then use samtools to convert, sort, and index each merged alignment: samtools sort produces a coordinate-sorted BAM, and samtools index produces the companion .bai file. Place the BAM and BAI in the BAM.25-35 results folder of the RiboDoc project volume. Sorting and indexing is required because htseq-count and riboWaltz expect position-ordered, indexed alignments.

## Related tools

- **samtools** (Sort, convert, and index the merged HISAT2/Bowtie2 alignments into BAM and BAI files) — https://sourceforge.net/projects/samtools/files/samtools/
- **RiboDoc** (Docker/Snakemake pipeline that orchestrates the sort-and-index step after genome alignment) — https://github.com/equipeGST/RiboDoc
- **HISAT2** (Produces the primary splice-aware SAM alignments that are merged and sorted) — http://daehwankimlab.github.io/hisat2/
- **Bowtie 2** (Remaps HISAT2-unaligned reads (≤1 mismatch); its SAM output feeds the merged, sorted BAM) — https://bowtie-bio.sourceforge.net/bowtie2/index.shtml

## Examples

```
samtools merge merged.sam hisat2.sam bowtie2.sam && samtools sort -o sorted.bam merged.sam && samtools index sorted.bam
```

## Evaluation signals

- Each sample yields a non-empty, valid BAM file in the BAM.25-35 results folder
- Each BAM has a companion .bai index file
- Sorted BAM is coordinate-ordered and readable by samtools quickcheck / samtools view
- Multimapping filter reports (Analysis_Table_summary) confirm only uniquely mapped reads are retained downstream
- htseq-count and riboWaltz/TRiP can consume the indexed BAM without errors

## Limitations

- Only reads left after rRNA/tRNA/viral RNA depletion and 25-35 nt length selection reach this step; low-quality samples must be removed and RiboDoc rerun
- Reads that remain unmapped after both HISAT2 and Bowtie2 are discarded, so very divergent or non-reference loci will be absent from the BAM
- RiboDoc expects a strict project-folder layout (lowercase subfolder names, FastQ files named [CONDITION]_[NAME].[REPLICATE].fastq.gz) for the BAM.25-35 outputs to be generated correctly

## Evidence

- [methods] Merge the HISAT2 and Bowtie2 SAM outputs into a single alignment per sample, discarding reads that remain unmapped so only uniquely usable mapped footprints are retained.: [evidence span withheld — anchor status: not_found]
- [methods] Convert, sort and index each merged alignment with samtools to produce BAM and BAI files in the BAM.25-35 results folder.: [evidence span withheld — anchor status: not_found]
- [methods] verify that each sample yields a non-empty, valid BAM file in BAM.25-35 together with its .bai index: [evidence span withheld — anchor status: not_found]
- [readme] Sort and index alignments with samtools: "Sort and index alignments with [`samtools`](https://sourceforge.net/projects/samtools/files/samtools/)"
- [methods] Reads not aligned in the first mapping step are remapped on the same fasta file with Bowtie2 (v2.4.2), using less strict default parameters that accept more complementarity errors (maximum of 1).: [evidence span withheld — anchor status: not_found]
