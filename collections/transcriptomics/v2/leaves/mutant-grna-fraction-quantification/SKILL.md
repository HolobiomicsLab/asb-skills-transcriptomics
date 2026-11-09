---
name: mutant-grna-fraction-quantification
description: Use when you need to estimate the fraction of mutant gRNA molecules in
  a CROP-seq/gRNA library BAM after consensus filtering that removes sequencing-error
  UMIs (min_reads threshold), to profile library quality or mutation-aware gRNA annotation.
when_to_use_negative:
- You only need cell-level gRNA-to-cell assignment counts — use assign_gRNA.assign_gRNA_to_cell
  with min_umi (default 3) instead of molecule-level mutant fraction quantification.
- Your goal is detecting CRISPR-Cas9 editing effects in the transcriptome (cutsite-spanning
  deletions), a separate GiRAFR analysis mode.
- You want to detect mutations in raw reads without consensus/error filtering — that
  inflates the mutant fraction with sequencing errors that min_reads=2 is designed
  to remove.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: GiRAFR
  role: Python package providing utils.gRNA_bam_filter, consensus_sequence.generate_consensus_sequence_gRNA,
    and variant.call_gRNA_variant used to filter BAMs, build UMI-cell barcode consensus
    sequences, and call mutations
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: pysam
  role: Backend over which GiRAFR reads and manipulates the SAM/BAM gRNA alignment
    files
  repo: https://github.com/pysam-developers/pysam
- name: samtools
  role: Used by GiRAFR's bam filtration step to split/filter the gRNA alignment bam
    file
- name: Python
  role: Implementation language; GiRAFR is installed with `pip install girafr` and
    run as python functions/scripts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:de6f20f0452fbe414b272316e90a6b7aacf84fb3ffd83a3bb42c64b85f3c5ebb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FunctionalEpigeneticsLab/GiRAFR
  tools:
  - GiRAFR
  - Python
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mutant-grna-fraction-quantification

## Summary

Quantifies the fraction of mutant gRNA molecules in a single-cell CRISPR screen gRNA library that remain after UMI consensus-sequence filtering (min_reads=2), using GiRAFR's bam filtering, consensus-sequence construction, and variant-calling modules. Mutant molecules that survive consensus filtering represent real library synthesis/viral-preparation mutations rather than sequencing errors.

## When to use

Use when you need to estimate the fraction of mutant gRNA molecules in a CROP-seq/gRNA library BAM after consensus filtering that removes sequencing-error UMIs (min_reads threshold), to profile library quality or mutation-aware gRNA annotation.

## When NOT to use

- You only need cell-level gRNA-to-cell assignment counts — use assign_gRNA.assign_gRNA_to_cell with min_umi (default 3) instead of molecule-level mutant fraction quantification.
- Your goal is detecting CRISPR-Cas9 editing effects in the transcriptome (cutsite-spanning deletions), a separate GiRAFR analysis mode.
- You want to detect mutations in raw reads without consensus/error filtering — that inflates the mutant fraction with sequencing errors that min_reads=2 is designed to remove.

## Inputs

- gRNA library BAM (e.g. gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam after GiRAFR bam filtering)
- CellRanger-corrected cell barcode list (filtered barcodes)
- Custom reference oligo pool FASTA: oligo_pool_plasmid.fa (from prepare.py with oligo_pool.csv, columns oligo_name and sequence, no header)
- Structure annotation: oligo_pool_plasmid_structure.gtf
- min_reads parameter (integer, default 1; set to 2 for sequencing-error filtering)

## Outputs

- consensus.sequence.gRNA.txt — most read-supported sequence per UMI-cell barcode combination after min_reads filtering
- CIGAR-like mutation strings per consensus gRNA sequence (from variant.call_gRNA_variant)
- Fraction of mutant gRNA molecules among surviving consensus sequences (28.4% for the in-house CROP-seq gRNA library)

## How to apply

First run GiRAFR's utils.gRNA_bam_filter (backed by samtools) on the gRNA library BAM to remove secondary alignments and reads not mapped to the designed gRNA cassette chromosomes, producing gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam. Then run consensus_sequence.generate_consensus_sequence_gRNA on the filtered BAM together with the CellRanger-corrected cell barcode list, taking the most read-supported sequence per UMI-cell barcode combination; set min_reads=2 so UMIs supported by fewer than 2 reads are discarded as sequencing errors, yielding consensus.sequence.gRNA.txt. Next call variants with variant.call_gRNA_variant by comparing each consensus sequence to oligo_pool_plasmid.fa annotated with oligo_pool_plasmid_structure.gtf, encoding variants in a CIGAR-like string format. Finally compute the fraction of consensus sequences carrying mutations — for the in-house CROP-seq gRNA library this was 28.4% of molecules, confirming mutant gRNA molecules are common in gRNA libraries.

## Related tools

- **GiRAFR** (Python package providing utils.gRNA_bam_filter, consensus_sequence.generate_consensus_sequence_gRNA, and variant.call_gRNA_variant used to filter BAMs, build UMI-cell barcode consensus sequences, and call mutations) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **pysam** (Backend over which GiRAFR reads and manipulates the SAM/BAM gRNA alignment files) — https://github.com/pysam-developers/pysam
- **samtools** (Used by GiRAFR's bam filtration step to split/filter the gRNA alignment bam file)
- **Python** (Implementation language; GiRAFR is installed with `pip install girafr` and run as python functions/scripts)

## Examples

```
variant.call_gRNA_variant(consensus_seq_file="consensus.sequence.gRNA.txt", ref_fasta="oligo_pool_plasmid.fa", structure_gtf="oligo_pool_plasmid_structure.gtf")  # with min_reads=2 upstream
```

## Evaluation signals

- The filtered BAM exists with the exact expected name: gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam (secondary alignments removed, only reads mapped to gRNA cassette chromosomes).
- consensus.sequence.gRNA.txt contains exactly one most-read-supported sequence per UMI-cell barcode combination, and no UMI supported by fewer than min_reads=2 reads remains.
- Variant calls are encoded in the CIGAR-like string format relative to oligo_pool_plasmid.fa using oligo_pool_plasmid_structure.gtf annotations.
- The computed mutant molecule fraction matches the reported 28.4% for the in-house CROP-seq gRNA library (same filtering and min_reads=2 settings).
- Setting min_reads=1 (default) should yield a higher raw mutant fraction, demonstrating the filtering effect attributable to sequencing errors.

## Limitations

- min_reads defaults to 1; the sequencing-error filtering effect depends on explicitly setting it to 2, so raw runs overstate the true mutant molecule fraction.
- A head-to-head comparison between published studies is unfeasible because studies use ad hoc customized pipelines and fixed thresholds.
- Design choices (multiplicity of infection, number of designed gRNAs, mutations inadvertently introduced in them, cells per perturbation) influence the measured mutant fraction.
- Fine-grained appraisal of which mutation types and locations are tolerable or damaging is beyond the scope of the study.
- The 28.4% figure was established for the in-house CROP-seq gRNA library; the fraction in other libraries may differ (public datasets showed up to 35% of cells transduced with a mutant gRNA).

## Evidence

- [methods] consensus_sequence.generate_consensus_sequence_gRNA: "consensus_sequence.generate_consensus_sequence_gRNA(bam_in, barcodes)"
- [methods] min_reads filter: "Integer number. By default = 1. UMIs with less than min_reads will be filtered out."
- [methods] filtered bam output name: "generate filtered bam file in output folder named as ``gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam``"
- [methods] consensus per UMI-cell barcode: "Step2: Construct consensus sequence for each UMI-Cell barcode combination"
- [intro] mutant molecules are common: "we show that mutant gRNA molecule are common in gRNA libraries and that, as a consequence, cells are often inaccurately annotated"
- [readme] GiRAFR purpose (README): "GiRAFR is a python package developed to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells in a sensitive, mutation-aware manner."
