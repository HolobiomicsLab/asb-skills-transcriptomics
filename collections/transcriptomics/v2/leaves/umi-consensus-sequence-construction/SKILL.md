---
name: umi-consensus-sequence-construction
description: Use when processing a single-cell CRISPR screen gRNA library BAM (mapped
  to the designed gRNA cassette reference) whose reads carry cell barcodes and UMIs,
  and you need per-UMI consensus sequences before calling mutations or assigning gRNAs
  to cells.
when_to_use_negative:
- The gRNA-to-cell annotation is already available from Cell Ranger feature barcoding
  and you only want intact-gRNA calls — this skill exists precisely because Cell Ranger
  misses mutant gRNAs.
- You are running GiRAFR's separate CRISPR-Cas9 editing-effect mode on transcriptome
  data, which does not use UMI consensus construction of the gRNA library.
- The input is raw unfiltered FASTQ or a BAM that still contains secondary alignments
  and reads not mapped to the designed gRNA cassette — run utils.gRNA_bam_filter first.
tools:
- name: GiRAFR
  role: Python package providing consensus_sequence.generate_consensus_sequence_gRNA
    for per-UMI consensus construction
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: pysam
  role: Underlying library GiRAFR uses to read and manipulate the gRNA-library BAM
  repo: https://github.com/pysam-developers/pysam
- name: samtools
  role: Used by utils.gRNA_bam_filter to produce the filtered BAM that this step consumes
provenance:
  source_task_ids:
  - task_005
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:f8c20a43ab4413ad1a84d2ed9a5a4951058f477729b332fe917f1bac6f15e54b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FunctionalEpigeneticsLab/GiRAFR
  tools:
  - GiRAFR
  - Python
  - pysam
  - samtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# UMI consensus sequence construction

## Summary

GiRAFR pipeline step 2: build one consensus gRNA sequence per UMI–cell barcode combination from the filtered gRNA-library BAM, retaining only UMIs supported by at least min_reads reads, so that subsequent mutation calling operates on collapsed rather than raw reads.

## When to use

Use when processing a single-cell CRISPR screen gRNA library BAM (mapped to the designed gRNA cassette reference) whose reads carry cell barcodes and UMIs, and you need per-UMI consensus sequences before calling mutations or assigning gRNAs to cells.

## When NOT to use

- The gRNA-to-cell annotation is already available from Cell Ranger feature barcoding and you only want intact-gRNA calls — this skill exists precisely because Cell Ranger misses mutant gRNAs.
- You are running GiRAFR's separate CRISPR-Cas9 editing-effect mode on transcriptome data, which does not use UMI consensus construction of the gRNA library.
- The input is raw unfiltered FASTQ or a BAM that still contains secondary alignments and reads not mapped to the designed gRNA cassette — run utils.gRNA_bam_filter first.

## Inputs

- Filtered gRNA-library BAM (gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam) with cell barcodes and UMIs in read tags
- Filtered cell barcode list (unzipped Cell Ranger corrected cell barcodes)
- min_reads parameter (integer, default = 1)

## Outputs

- Consensus sequence file, one most-supported gRNA sequence per UMI-cell barcode combination (consensus.bam / consensus sequence input for variant.call_gRNA_variant)

## How to apply

After step 1 (utils.gRNA_bam_filter) has produced gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam, pass this filtered BAM together with the filtered barcode list to consensus_sequence.generate_consensus_sequence_gRNA(bam_in, barcodes). For each UMI–cell barcode combination, take the most read-supported sequence as the consensus, keeping only UMIs supported by at least min_reads reads (default = 1; raise this to suppress spurious UMIs at the cost of sensitivity). The rationale is that raw reads over-count molecules: collapsing reads to one sequence per UMI yields the true number of gRNA molecules per cell, which GiRAFR's assign_gRNA.assign_gRNA_to_cell step (min_umi = 3 or auto Gaussian-mixture thresholding) depends on. The consensus output feeds directly into variant.call_gRNA_variant, which compares each consensus to the oligo_pool_plasmid.fa reference and annotates mutation positions against oligo_pool_plasmid_structure.gtf as CIGAR-like variant strings.

## Related tools

- **GiRAFR** (Python package providing consensus_sequence.generate_consensus_sequence_gRNA for per-UMI consensus construction) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **pysam** (Underlying library GiRAFR uses to read and manipulate the gRNA-library BAM) — https://github.com/pysam-developers/pysam
- **samtools** (Used by utils.gRNA_bam_filter to produce the filtered BAM that this step consumes)

## Examples

```
girafr gRNA_mutation -f ConfigFile
```

## Evaluation signals

- The consensus output contains exactly one most-supported sequence per UMI–cell barcode combination, and UMIs with fewer than min_reads reads are absent.
- Downstream variant.call_gRNA_variant succeeds on the consensus file, producing consensus.sequence.gRNA.variant.txt with variants encoded in CIGAR-like string format relative to oligo_pool_plasmid.fa.
- Per-cell UMI counts derived from the consensus output (median ~5, ~16, ~31 at 34%, 59%, 92% sequencing saturation in the in-house CROP-seq reproduction) fall in the expected range from Figure 3b.
- cells.gRNA.txt and gRNA.umi.threshold.txt produced afterwards match the documented GiRAFR output format.
- Mutant gRNA frequencies per experiment remain within the reported 2–35% range when the consensus step feeds the full 26-experiment public-dataset analysis.

## Limitations

- min_reads defaults to 1, so single-read UMIs are retained by default; at low sequencing depth this can incorporate synthesis- or sequencing-error sequences into consensuses.
- Specific defined read depths need to be attained for accurate gRNA assignment and mutation detection; saturation below ~34% yields substantially fewer UMIs per cell and lower annotation accuracy.
- The consensus quality is bounded by the quality of the upstream gRNA BAM filtration; unfiltered BAMs with secondary or off-cassette alignments will contaminate consensuses.

## Evidence

- [methods] Step2: Construct consensus sequence for each UMI-Cell barcode combination: "Step2: Construct consensus sequence for each UMI-Cell barcode combination"
- [methods] min_reads default: "Integer number. By default = 1. UMIs with less than min_reads will be filtered out."
- [discussion] read depths needed: "For example, specific defined read depths need to be attained for accurate gRNA assignment and mutation detection."
- [readme] GiRAFR package description: "GiRAFR is a python package developed to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells in a sensitive, mutation-aware manner."
