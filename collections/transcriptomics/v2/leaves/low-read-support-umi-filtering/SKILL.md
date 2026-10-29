---
name: low-read-support-umi-filtering
description: Use when quantifying mutant gRNA molecule fractions in a single-cell
  CRISPR screen gRNA library and sequencing-error-derived molecules must be excluded
  before variant calling.
when_to_use_negative:
- You want to profile ambient or very-low-abundance gRNA capture rather than remove
  low-support UMIs — dropping UMIs below min_reads biases the measured molecule pool
  toward well-supported sequences
- The goal is assigning gRNAs to cells (assign_gRNA.assign_gRNA_to_cell with min_umi,
  default 3), which is a distinct downstream step rather than molecule-level error
  filtering
- Detecting CRISPR-Cas9-induced DNA editing effects in transcriptome data, which uses
  GiRAFR's separate editing-effect analysis mode
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: GiRAFR
  role: Python package providing utils.gRNA_bam_filter, consensus_sequence.generate_consensus_sequence_gRNA
    and variant.call_gRNA_variant for the filtering and mutation-calling workflow
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: pysam
  role: Underlying library GiRAFR is based on for reading and manipulating the SAM/BAM
    gRNA alignments
  repo: https://github.com/pysam-developers/pysam
- name: samtools
  role: Used by utils.gRNA_bam_filter to split the alignment file into filtered BAM
    files
- name: Cell Ranger
  role: Source of the cell barcodes used to filter which UMI-cell barcode combinations
    enter consensus building
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:9a694f21f7c445176d68e3ce459c880e018d32cfbb19a3aebdc826b9c9aee272
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FunctionalEpigeneticsLab/GiRAFR
  tools:
  - Cell Ranger
  - GiRAFR
  - Python
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# low-read-support-umi-filtering

## Summary

Discard sequencing-error-derived gRNA molecules by filtering UMI-cell barcode consensus sequences with fewer than a minimum read count (GiRAFR's min_reads parameter), before calling gRNA mutations; for the in-house CROP-seq library this still left 28.4% of molecules mutant.

## When to use

Use when quantifying mutant gRNA molecule fractions in a single-cell CRISPR screen gRNA library and sequencing-error-derived molecules must be excluded before variant calling.

## When NOT to use

- You want to profile ambient or very-low-abundance gRNA capture rather than remove low-support UMIs — dropping UMIs below min_reads biases the measured molecule pool toward well-supported sequences
- The goal is assigning gRNAs to cells (assign_gRNA.assign_gRNA_to_cell with min_umi, default 3), which is a distinct downstream step rather than molecule-level error filtering
- Detecting CRISPR-Cas9-induced DNA editing effects in transcriptome data, which uses GiRAFR's separate editing-effect analysis mode

## Inputs

- gRNA library BAM file (single-cell CRISPR screen, e.g. CROP-seq)
- filtered CellRanger-corrected cell barcode list
- oligo_pool_plasmid.fa reference of designed gRNA sequences
- oligo_pool_plasmid_structure.gtf annotation
- GiRAFR ConfigFile with min_reads parameter (set to 2)

## Outputs

- gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam (filtered BAM)
- consensus.sequence.gRNA.txt (consensus sequence per UMI-cell barcode after min_reads=2 filtering)
- per-molecule mutation calls in CIGAR-like string format
- fraction of mutant gRNA molecules (e.g. 28.4% for the in-house CROP-seq gRNA library)

## How to apply

First run GiRAFR's utils.gRNA_bam_filter (backed by samtools) on the gRNA library BAM to remove secondary alignments and reads not mapped to the designed gRNA cassette chromosomes, producing gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam. Then run consensus_sequence.generate_consensus_sequence_gRNA on that filtered BAM together with the CellRanger-corrected cell barcode list, taking the most read-supported sequence per UMI-cell barcode combination and setting min_reads=2 so UMIs supported by fewer than 2 reads are discarded as sequencing errors, yielding consensus.sequence.gRNA.txt. Next call mutations with variant.call_gRNA_variant against oligo_pool_plasmid.fa with oligo_pool_plasmid_structure.gtf annotation, encoding variants in a CIGAR-like string format. Finally compute the fraction of mutant molecules among surviving consensus sequences; the rationale is that low-read-support UMIs are most plausibly PCR/sequencing errors rather than true library variants.

## Related tools

- **GiRAFR** (Python package providing utils.gRNA_bam_filter, consensus_sequence.generate_consensus_sequence_gRNA and variant.call_gRNA_variant for the filtering and mutation-calling workflow) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **pysam** (Underlying library GiRAFR is based on for reading and manipulating the SAM/BAM gRNA alignments) — https://github.com/pysam-developers/pysam
- **samtools** (Used by utils.gRNA_bam_filter to split the alignment file into filtered BAM files)
- **Cell Ranger** (Source of the cell barcodes used to filter which UMI-cell barcode combinations enter consensus building)

## Examples

```
python -m girafr consensus_sequence.generate_consensus_sequence_gRNA gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam barcodes.txt --min_reads 2
```

## Evaluation signals

- Filtered BAM is named gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam and contains no secondary alignments or off-cassette-chromosome reads
- Every retained consensus sequence is supported by at least min_reads=2 reads for its UMI-cell barcode combination
- Fraction of mutant gRNA molecules after filtering reproduces the reported 28.4% for the in-house CROP-seq gRNA library
- Variant calls are encoded in the CIGAR-like string format relative to oligo_pool_plasmid.fa with oligo_pool_plasmid_structure.gtf annotation
- Filtered publicly available datasets remain consistent with the paper's finding that up to ~34–35% of cells carry a mutant gRNA

## Limitations

- min_reads=2 removes only the low-support tail; a large fraction of molecules (28.4% in-house, up to ~35% of cells in public datasets) remain mutant, indicating most mutations are real library/virus-prep variants, not sequencing errors
- Default min_reads is 1, so the filtering must be explicitly configured; default min_umi is 3 for the separate cell-assignment step
- The paper does not appraise which gRNA mutation types or locations are tolerable or damaging, declared beyond its scope

## Evidence

- [methods] min_reads: "Integer number. By default = 1. UMIs with less than min_reads will be filtered out."
- [methods] consensus step: "Step2: Construct consensus sequence for each UMI-Cell barcode combination"
- [methods] filtered bam output: "generate filtered bam file in output folder named as ``gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam``"
- [methods] mutation calling: "variant.call_gRNA_variant(consensus_seq_file, ref_fasta, structure_gtf)"
- [readme] readme purpose: "GiRAFR is a python package developed to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells in a sensitive, mutation-aware manner."
