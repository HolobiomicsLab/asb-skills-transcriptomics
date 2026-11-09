---
name: unaligned-read-remapping-with-bowtie2
description: Use when a Ribo-seq pipeline leaves reads unmapped after a first splice-aware
  genome alignment (HISAT2) and you want to salvage them with a second, less stringent
  ungapped mapper like Bowtie2.
when_to_use_negative:
- Reads are short contaminant RNAs (rRNA, tRNA, viral RNA) to be discarded — those
  are removed in a separate depletion step with Bowtie2, not remapped to the genome
- Your aligner already captures splice junctions and you require gapped alignment
  tolerance for the second pass; the remap is deliberately less strict (max 1 mismatch),
  which is unsuitable if you need high-fidelity alignments
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: HISAT2
  role: First-pass splice-aware alignment of 25-35 nt footprints to the reference
    genome, capturing reads spanning exon junctions; its unaligned output is the input
    to the remap
  repo: http://daehwankimlab.github.io/hisat2/
- name: Bowtie 2
  role: Second-pass remapping of HISAT2-unaligned reads against the same FASTA (v2.4.2)
    with less strict parameters accepting at most 1 complementarity error
  repo: https://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: samtools
  role: Convert, sort and index the merged HISAT2/Bowtie2 alignments to produce BAM
    and BAI files
  repo: https://sourceforge.net/projects/samtools/files/samtools/
- name: RiboDoc
  role: Docker/Snakemake pipeline orchestrating the double-mapping step and downstream
    BAM outputs
  repo: https://github.com/equipeGST/RiboDoc
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:82b46e64b67b429f175246d2dcaa0bbe777f10340c9904fd984aa16438a9723d
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Unaligned read remapping with Bowtie2

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

In RiboDoc's double-mapping step, ribosome footprints that HISAT2 fails to align splice-aware are remapped against the same genome FASTA with Bowtie2 (v2.4.2) using less strict parameters (at most 1 mismatch), recovering reads on complex genomes and maximizing mapped footprints before BAM generation.

## When to use

Use when a Ribo-seq pipeline leaves reads unmapped after a first splice-aware genome alignment (HISAT2) and you want to salvage them with a second, less stringent ungapped mapper like Bowtie2.

## When NOT to use

- Reads are short contaminant RNAs (rRNA, tRNA, viral RNA) to be discarded — those are removed in a separate depletion step with Bowtie2, not remapped to the genome
- Your aligner already captures splice junctions and you require gapped alignment tolerance for the second pass; the remap is deliberately less strict (max 1 mismatch), which is unsuitable if you need high-fidelity alignments

## Inputs

- rRNA-depleted, length-filtered FastQ reads (25-35 nt RPF selection) in gzip-compressed format
- pre-built reference genome index (e.g. from the annex_database/index_files folder)
- reference genome FASTA (e.g. Saccharomyces_cerevisiae.R64-1-1.dna.toplevel.fa.gz)

## Outputs

- Merged, uniquely-mapped alignments per sample (SAM converted to sorted, indexed BAM)
- BAM.25-35 results folder containing non-empty BAM files with .bai indexes
- Analysis_Table_summary outputs reporting uniquely mapped reads after multimapping filtering

## How to apply

After rRNA depletion and 25-35 nt RPF length filtering, align the depleted FastQ reads to a pre-built reference genome index with HISAT2 in splice-aware mode so footprints spanning exon junctions are captured. Collect the reads left unaligned by HISAT2 and remap them against the same reference FASTA with Bowtie2 (v2.4.2), whose default parameters accept more complementarity errors with a maximum of 1 mismatch. Merge the HISAT2 and Bowtie2 SAM outputs into a single alignment per sample, discarding reads that remain unmapped so only uniquely usable mapped footprints are retained, and remove multimapping reads to keep uniquely mapped ones. Finally, convert, sort and index each merged alignment with samtools to produce BAM and BAI files in the BAM.25-35 results folder.

## Related tools

- **HISAT2** (First-pass splice-aware alignment of 25-35 nt footprints to the reference genome, capturing reads spanning exon junctions; its unaligned output is the input to the remap) — http://daehwankimlab.github.io/hisat2/
- **Bowtie 2** (Second-pass remapping of HISAT2-unaligned reads against the same FASTA (v2.4.2) with less strict parameters accepting at most 1 complementarity error) — https://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **samtools** (Convert, sort and index the merged HISAT2/Bowtie2 alignments to produce BAM and BAI files) — https://sourceforge.net/projects/samtools/files/samtools/
- **RiboDoc** (Docker/Snakemake pipeline orchestrating the double-mapping step and downstream BAM outputs) — https://github.com/equipeGST/RiboDoc

## Evaluation signals

- Each sample yields a non-empty, valid BAM file in the BAM.25-35 folder together with its .bai index
- The multimapping filter step has kept uniquely mapped reads as reported in the Analysis_Table_summary outputs
- Reads that remain unmapped after both HISAT2 and Bowtie2 are discarded, so the merged alignment contains only mapped footprints
- The Bowtie2 remap enforces a maximum of 1 mismatch per alignment, consistent with the pipeline's less strict default parameters

## Limitations

- Bowtie2's remap accepts at most 1 mismatch, so reads with more complementarity errors stay unmapped and are discarded
- The example GEO GSE173856 dataset does not align enough material for use of the riboWaltz pipeline, illustrating that low alignment yield can propagate downstream
- Step parameters depend on a precise project folder architecture (fastq subfolder, indexed fasta in annex_database/index_files) and lowercase subfolder names

## Evidence

- [methods] Reads not aligned in the first mapping step are remapped on the same fasta file with Bowtie2 (v2.4.2), using less strict default parameters that accept more complementarity errors (maximum of 1).: [evidence span withheld — anchor status: not_found]
