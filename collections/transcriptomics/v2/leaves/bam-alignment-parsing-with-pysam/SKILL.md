---
name: bam-alignment-parsing-with-pysam
description: Use when you have a gRNA-aligned BAM file (e.g. gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam)
  plus a filtered cell barcode list and need per-UMI consensus gRNA sequences for
  mutation calling.
when_to_use_negative:
- The goal is detecting CRISPR-Cas9-induced DNA editing in transcriptome data — that
  is GiRAFR's separate editing_effect analysis mode, not the gRNA consensus/mutation
  pipeline.
- The input is already a per-UMI consensus table or a per-cell gRNA assignment matrix
  (proceed to assign_gRNA.assign_gRNA_to_cell instead).
- The input is raw FASTQ rather than an aligned BAM — alignment must happen before
  pysam-based BAM filtration applies.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: pysam
  role: Python module for reading and manipulating SAM/BAM alignment files; GiRAFR
    is built on it to filter gRNA BAM reads and extract UMI/barcode records
  repo: https://github.com/pysam-developers/pysam
- name: GiRAFR
  role: pysam-based package whose gRNA_mutation pipeline consumes the parsed BAM to
    build per-UMI consensus sequences and call gRNA variants
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: samtools
  role: Used to split the alignment BAM file into per-gRNA BAM files during the filtration
    step
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:cc9c8c739bb85999dbb16f959de7ecc15c9de9b9d92afa5ca4e53b2b164ebd4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pysam-developers/pysam
  tools:
  - GiRAFR
  - Python
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# bam-alignment-parsing-with-pysam

## Summary

Read and manipulate gRNA alignment BAM files with pysam (a lightweight wrapper of the samtools C-API) to filter reads by cell barcode, group by UMI + cell barcode, and resolve per-UMI consensus gRNA sequences for mutation-aware single-cell CRISPR screen annotation.

## When to use

Use when you have a gRNA-aligned BAM file (e.g. gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam) plus a filtered cell barcode list and need per-UMI consensus gRNA sequences for mutation calling.

## When NOT to use

- The goal is detecting CRISPR-Cas9-induced DNA editing in transcriptome data — that is GiRAFR's separate editing_effect analysis mode, not the gRNA consensus/mutation pipeline.
- The input is already a per-UMI consensus table or a per-cell gRNA assignment matrix (proceed to assign_gRNA.assign_gRNA_to_cell instead).
- The input is raw FASTQ rather than an aligned BAM — alignment must happen before pysam-based BAM filtration applies.

## Inputs

- gRNA alignment BAM file (e.g. gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam)
- Unzipped filtered cell barcode list
- Reference cassette FASTA (oligo_pool_plasmid.fa) from prepare.py
- Structure annotation GTF (oligo_pool_plasmid_structure.gtf)
- GiRAFR ConfigFile (min_reads, min_umi parameters)

## Outputs

- consensus.sequence.gRNA.txt
- consensus.seqeunce.gRNA.all_umi.txt
- consensus.bam
- Non-consensus.bam
- consensus.sequence.gRNA.variant.txt (semicolon-separated structure annotations with CIGAR-like mutation strings)
- all.MT.txt (mutation-detail file)

## How to apply

Use pysam to open the filtered gRNA BAM file and retain only reads whose cell barcodes appear in the unzipped filtered barcode list, producing the filtered file gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam (utils.gRNA_bam_filter). Then group reads by UMI + cell barcode combination and take the most read-supported sequence as the consensus for each UMI (consensus_sequence.generate_consensus_sequence_gRNA). Apply the min_reads filter — the paper's pipeline uses 2 reads per UMI to exclude sequencing errors, though the tool default in ConfigFile is 1 — before emitting consensus outputs. Finally pass consensus.bam / the consensus sequence file to variant.call_gRNA_variant with the reference cassette oligo_pool_plasmid.fa and oligo_pool_plasmid_structure.gtf to encode mutations as a CIGAR-like string (digits = exact matches, nucleotides = mutated bases, bracketed counts for insertion/deletion/substitution events, hard clips excluded).

## Related tools

- **pysam** (Python module for reading and manipulating SAM/BAM alignment files; GiRAFR is built on it to filter gRNA BAM reads and extract UMI/barcode records) — https://github.com/pysam-developers/pysam
- **GiRAFR** (pysam-based package whose gRNA_mutation pipeline consumes the parsed BAM to build per-UMI consensus sequences and call gRNA variants) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **samtools** (Used to split the alignment BAM file into per-gRNA BAM files during the filtration step)

## Examples

```
consensus_sequence.generate_consensus_sequence_gRNA(bam_in, barcodes)
```

## Evaluation signals

- consensus.sequence.gRNA.variant.txt exists and every mutation annotation is a well-formed CIGAR-like string (no hard-clip tokens; insertion/deletion counts match the bracketed digits).
- Mutation event positions in the CIGAR-like string are consistent with the structures annotated in oligo_pool_plasmid_structure.gtf (digit = distance to structure start).
- No UMI with fewer than min_reads reads (2 in the paper's setting) contributes a consensus sequence; UMIs below this threshold appear only in the all_umi output or are discarded.
- All consensus records originate from reads whose cell barcodes are present in the unzipped filtered barcode list.
- consensus.bam and Non-consensus.bam together partition the barcode-filtered reads.

## Limitations

- Accurate gRNA assignment and mutation detection requires specific defined read depths to be attained; with low per-cell UMI counts, detection rates and cell assignment accuracy suffer.
- The default min_reads = 1 in ConfigFile does not exclude sequencing errors; the paper's pipeline raises it to 2, and the threshold is a configurable decision point.
- Whether inter-experiment differences in average gRNA UMIs stem from low gRNA expression or low detection rates is undetermined, which impacts ambient vs endogenous and singlet vs multiplet discrimination.

## Evidence

- [intro] GiRAFR (Guide RNA Anomaly and Functionality Revealer), a tool based on pysam: "GiRAFR (Guide RNA Anomaly and Functionality Revealer), a tool based on pysam (https://github.com/pysam-developers/pysam)"
- [methods] Step2: Construct consensus sequence for each UMI-Cell barcode combination: "Step2: Construct consensus sequence for each UMI-Cell barcode combination"
- [methods] utils.gRNA_bam_filter(filename, samtools_path): "utils.gRNA_bam_filter(filename, samtools_path)"
- [methods] Integer number. By default = 1. UMIs with less than min_reads will be filtered out.: "Integer number. By default = 1. UMIs with less than min_reads will be filtered out."
- [readme] GiRAFR is a python package developed to perform quality control of single cell CRISPR screens: "GiRAFR is a python package developed to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells in a sensitive, mutation-aware manner."
