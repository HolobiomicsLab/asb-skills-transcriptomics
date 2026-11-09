---
name: splice-aware-ribosome-footprint-genome-alignment
description: Use when aligning rRNA-depleted ribosome-footprint reads (e.g. 25-35
  nt RPFs) to a genome whose transcripts contain exon junctions, requiring splice-aware
  mapping followed by a less strict remapping of unaligned reads.
when_to_use_negative:
- Reads still contain contaminant rRNA/tRNA — run the Bowtie2 contaminant-depletion
  step first, as aligned contaminants would pollute the genome mapping
- Reads have not been adapter-trimmed and length-filtered to 25-35 nt with Cutadapt
  — unfiltered read-length distributions break the RPF assumptions of the BAM.25-35
  output
- Aligning transcriptomes without exon junctions where splice-aware mapping is unnecessary
  and Bowtie2 alone suffices
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: HISAT2
  role: first-pass splice-aware alignment of depleted RPF reads to the reference genome
    FASTA to capture footprints spanning exon junctions
- name: Bowtie 2
  role: remaps (v2.4.2) the reads unaligned by HISAT2 against the same FASTA with
    less strict defaults accepting a maximum of 1 mismatch
  repo: https://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: samtools
  role: converts, sorts and indexes the merged HISAT2/Bowtie2 alignments into BAM
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
  source_content_hash: sha256:98216f80657198acb7b643967d426beea6e08a77a9a612187abc2b3d613969d3
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
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# splice-aware-ribosome-footprint-genome-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

RiboDoc's double-mapping step: rRNA-depleted, 25-35 nt RPF FastQ reads are first aligned to the reference genome with HISAT2 in splice-aware mode to recover footprints spanning exon junctions, then reads left unaligned by HISAT2 are remapped against the same FASTA with Bowtie2 (v2.4.2) permitting at most 1 mismatch; merged, sorted and indexed SAM/BAM outputs are produced with samtools.

## When to use

Use when aligning rRNA-depleted ribosome-footprint reads (e.g. 25-35 nt RPFs) to a genome whose transcripts contain exon junctions, requiring splice-aware mapping followed by a less strict remapping of unaligned reads.

## When NOT to use

- Reads still contain contaminant rRNA/tRNA — run the Bowtie2 contaminant-depletion step first, as aligned contaminants would pollute the genome mapping
- Reads have not been adapter-trimmed and length-filtered to 25-35 nt with Cutadapt — unfiltered read-length distributions break the RPF assumptions of the BAM.25-35 output
- Aligning transcriptomes without exon junctions where splice-aware mapping is unnecessary and Bowtie2 alone suffices

## Inputs

- rRNA-depleted, adapter-trimmed and length-filtered FastQ reads (25-35 nt RPF selection), gzipped
- pre-built HISAT2/Bowtie2 genome index built from the reference genome FASTA (e.g. Saccharomyces_cerevisiae.R64-1-1.dna.toplevel.fa from Ensembl), stored in annex_database/index_files

## Outputs

- Merged, sorted BAM file per sample in the BAM.25-35 results folder
- Corresponding .bai index per sample
- Alignment counts fed to the Analysis_Table_summary (raw reads / filtered reads / multimapping / unique map)

## How to apply

Load the rRNA-depleted, length-filtered FastQ reads (25-35 nt RPF selection) together with the pre-built genome index from the annex_database/index_files folder, and align them to the reference genome FASTA with HISAT2 in splice-aware mode so footprints spanning exon junctions are captured. Collect the reads left unaligned by HISAT2 and remap them against the same reference FASTA with Bowtie2 v2.4.2 using less strict default parameters that accept more complementarity errors (maximum of 1). Merge the HISAT2 and Bowtie2 SAM outputs into a single alignment per sample, discarding reads that remain unmapped so only uniquely usable mapped footprints are retained. Finally convert, sort and index each merged alignment with samtools to produce BAM and BAI files in the BAM.25-35 results folder; the downstream multimapping filter step keeps uniquely mapped reads.

## Related tools

- **HISAT2** (first-pass splice-aware alignment of depleted RPF reads to the reference genome FASTA to capture footprints spanning exon junctions)
- **Bowtie 2** (remaps (v2.4.2) the reads unaligned by HISAT2 against the same FASTA with less strict defaults accepting a maximum of 1 mismatch) — https://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **samtools** (converts, sorts and indexes the merged HISAT2/Bowtie2 alignments into BAM and BAI files) — https://sourceforge.net/projects/samtools/files/samtools/
- **RiboDoc** (Docker/Snakemake pipeline orchestrating the double-mapping step and downstream BAM outputs) — https://github.com/equipeGST/RiboDoc

## Evaluation signals

- Each sample yields a non-empty, valid BAM file in the BAM.25-35 folder together with its .bai index
- The multimapping filter step has kept uniquely mapped reads, as reported in the Analysis_Table_summary outputs (Raw reads / Filtered reads / Multimapping / Unique map)
- Bowtie2 remapping is restricted to reads unaligned by HISAT2 and uses the maximum-1-mismatch setting
- Merged alignments are sorted and indexed (samtools BAI present) before read counting with htseq-count

## Limitations

- Bowtie2 remapping tolerates at most 1 complementarity error, so reads with more mismatches remain unmapped and are discarded
- Reads that remain unmapped after both HISAT2 and Bowtie2 are dropped, so only uniquely usable mapped footprints are retained
- The example GEO GSE173856 dataset 'does not align enough material for a use of the riboWaltz pipeline' — low-aligning inputs may starve downstream steps

## Evidence

- [methods] workflow: [evidence span withheld — anchor status: not_found]
- [methods] finding: [evidence span withheld — anchor status: not_found]
