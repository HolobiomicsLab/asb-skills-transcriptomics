---
name: mutation-aware-grna-variant-calling
description: Use when analyzing CROP-seq/single-cell CRISPR screen gRNA library alignments
  (BAM) where mutant gRNAs need to be detected and assigned to cells, e.g. quantifying
  cells lacking a gRNA call under standard feature barcoding pipelines.
when_to_use_negative:
- The goal is detecting CRISPR-Cas9 editing effects in transcriptome data — GiRAFR
  exposes this in a separate analysis mode (gRNA_mutation mode is for gRNA library
  mutation profiling)
- Cell barcodes have not been corrected/produced by Cell Ranger, so the filtered_barcode
  input cannot be supplied
- Only intact gRNA identity per cell is needed and mutation-awareness is irrelevant
  — a simpler Cell Ranger feature-counting approach may suffice, at the cost of missing
  mutant gRNAs
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3170
tools:
- name: GiRAFR
  role: 'Main pipeline: BAM filtering, UMI-cell consensus building, variant calling,
    and mutation-aware gRNA-to-cell assignment'
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: pysam
  role: Python SAM/BAM backend on which GiRAFR is built for reading/manipulating gRNA
    library alignments
  repo: https://github.com/pysam-developers/pysam
- name: samtools
  role: Used by utils.gRNA_bam_filter to split/filter the gRNA library alignment file
- name: CellRanger
  role: Produces the corrected cell barcodes used as filtered_barcode input; serves
    as the baseline annotation for comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:17fd7c3a57f306f86473bec06341d29069d9bbfe5829ef29f49e3300aa1350d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FunctionalEpigeneticsLab/GiRAFR
  tools:
  - CellRanger
  - GiRAFR
  - Python
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mutation-aware-grna-variant-calling

## Summary

Calling intact and mutant gRNA sequences in single-cell CRISPR screens by building per-UMI consensus sequences from the gRNA library BAM and comparing them to the designed oligo pool reference, then assigning gRNAs to cells with a mutation-aware UMI threshold. This catches mutant gRNAs that Cell Ranger feature barcoding misses.

## When to use

Use when analyzing CROP-seq/single-cell CRISPR screen gRNA library alignments (BAM) where mutant gRNAs need to be detected and assigned to cells, e.g. quantifying cells lacking a gRNA call under standard feature barcoding pipelines.

## When NOT to use

- The goal is detecting CRISPR-Cas9 editing effects in transcriptome data — GiRAFR exposes this in a separate analysis mode (gRNA_mutation mode is for gRNA library mutation profiling)
- Cell barcodes have not been corrected/produced by Cell Ranger, so the filtered_barcode input cannot be supplied
- Only intact gRNA identity per cell is needed and mutation-awareness is irrelevant — a simpler Cell Ranger feature-counting approach may suffice, at the cost of missing mutant gRNAs

## Inputs

- gRNA library alignment BAM mapped to the custom gRNA-cassette reference
- Cell Ranger corrected cell barcodes (unzipped filtered_barcode file)
- gRNA library reference FASTA (oligo_pool_plasmid.fa)
- gRNA cassette structure annotation GTF (oligo_pool_plasmid_structure.gtf)
- samtools installation for BAM filtering

## Outputs

- Filtered BAM: gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam
- Per-UMI-cell consensus sequences (consensus.bam)
- Variant calls in CIGAR-like string format: consensus.sequence.gRNA.variant.txt
- Per-cell gRNA assignments including mutant gRNA calls

## How to apply

Build a custom reference from the oligo pool (oligo_pool_plasmid.fa plus oligo_pool_plasmid_structure.gtf, generated with prepare.py) and configure ConfigFile with gRNA_bam_file, filtered_barcode (unzipped Cell Ranger corrected cell barcodes), ref_fasta and structure_gtf, then run 'girafr gRNA_mutation -f ConfigFile'. Filter the gRNA library BAM with utils.gRNA_bam_filter via samtools, removing secondary alignments and reads not mapped to the designed gRNA cassette to produce gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam. Construct a consensus sequence for each UMI-cell barcode combination with consensus_sequence.generate_consensus_sequence_gRNA (most read-supported sequence per UMI, min_reads default = 1). Call mutations with variant.call_gRNA_variant against ref_fasta, annotating positions with the structure GTF and writing consensus.sequence.gRNA.variant.txt in a CIGAR-like string format. Assign gRNAs to cells with assign_gRNA.assign_gRNA_to_cell using auto = True (per-gRNA two-component Gaussian mixture UMI threshold fitting, pool = True to pool intact and mutated UMIs) with min_umi = 3 as the fixed fallback threshold; optionally add mutation details via assign_gRNA.add_variant_type and profile_MT_pattern.py.

## Related tools

- **GiRAFR** (Main pipeline: BAM filtering, UMI-cell consensus building, variant calling, and mutation-aware gRNA-to-cell assignment) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **pysam** (Python SAM/BAM backend on which GiRAFR is built for reading/manipulating gRNA library alignments) — https://github.com/pysam-developers/pysam
- **samtools** (Used by utils.gRNA_bam_filter to split/filter the gRNA library alignment file)
- **CellRanger** (Produces the corrected cell barcodes used as filtered_barcode input; serves as the baseline annotation for comparison)

## Examples

```
girafr gRNA_mutation -f ConfigFile
```

## Evaluation signals

- Filtered BAM is named gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam, confirming secondary and off-cassette alignments were removed
- consensus.sequence.gRNA.variant.txt contains variants encoded in the CIGAR-like string format
- Per-cell assignment counts reproduce benchmark numbers: 2317/5744 cells without a gRNA call by Cell Ranger, and 1198/5744 cells (20.7%) assigned a mutant gRNA by GiRAFR, including 481 cells previously assigned no gRNA
- Cells with UMI counts >= min_umi (default 3) are assigned a gRNA; the auto Gaussian-mixture threshold is per-gRNA with intact and mutated UMIs pooled
- UMIs with fewer than min_reads reads (default 1) are excluded from consensus construction

## Limitations

- Fixed ad hoc thresholds across studies render head-to-head comparisons between published studies unfeasible; the Gaussian-mixture auto mode addresses this but depends on UMI distributions
- Dramatic inter-experiment differences in average gRNA UMIs (low gRNA expression vs low detection rates) can detrimentally impact accuracy of assigning gRNAs to cells and discriminating ambient from endogenous expression
- Design choices (multiplicity of infection, number of designed gRNAs, mutations inadvertently introduced, cells per perturbation) influence assignment accuracy and require parameter finetuning
- Fine-grained appraisal of which gRNA mutation types and locations are tolerable or damaging was declared beyond the scope of the study

## Evidence

- [abstract] utility: "We therefore developed a python tool to identify and characterize intact and mutant gRNAs, called GiRAFR."
- [methods] filter step: "utils.gRNA_bam_filter(filename, samtools_path)"
- [methods] consensus step: "consensus_sequence.generate_consensus_sequence_gRNA(bam_in, barcodes)"
- [methods] variant step: "variant.call_gRNA_variant(consensus_seq_file, ref_fasta, structure_gtf)"
- [methods] thresholds: "Integer number. By default = 3. Cells with UMIs >= min_umi will be assigned with gRNA."
- [intro] scope: "GiRAFR ... to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells in a sensitive, mutation-aware manner."
- [methods] workflow ordering: "step 1: gRNA bam file filtration."
- [readme] install: "GiRAFR is a python package developed to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells in a sensitive, mutation-aware manner."
