---
name: grna-variant-calling-with-cigar-like-encoding
description: Use when single-cell CRISPR screen gRNA alignments need mutation-aware
  calling — i.e., you have a consensus.bam of UMI-cell barcode consensus sequences
  and an oligo_pool_plasmid.fa reference, and must distinguish intact from mutant
  gRNAs before assigning gRNAs to cells.
when_to_use_negative:
- The analysis goal is CRISPR-Cas9 editing-effect detection in transcriptome data
  (deletions passing cutsite) rather than gRNA library mutation profiling — GiRAFR
  provides a separate analysis mode for that.
- Input is a gene-expression matrix or an already-computed gRNA-to-cell assignment
  table; variant calling is upstream of annotation and cannot recover mutation information
  from aggregated outputs.
- No plasmid oligo pool reference (oligo_pool_plasmid.fa / structure.gtf) is available,
  since consensus sequences must be compared to the designed gRNA reference to call
  variants.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: GiRAFR
  role: Python/pysam package whose variant module (variant.call_gRNA_variant) performs
    the mutation calling against the oligo pool reference and structure GTF; also
    provides bam filtering, consensus construction, and gRNA-to-cell assignment
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: pysam
  role: Python wrapper of the samtools C-API used to read and manipulate the SAM/BAM
    gRNA alignments underlying variant calling
  repo: https://github.com/pysam-developers/pysam
- name: samtools
  role: Splits and filters the gRNA bam file, removing secondary alignments and off-cassette
    alignments before consensus construction and variant calling
- name: Python
  role: Implementation language for GiRAFR's variant-calling and annotation workflow
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:d91ca5334d5e3494f455c0484ba91d75b0d8152c2580c307aa0144bbb8d8a279
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
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# grna-variant-calling-with-cigar-like-encoding

## Summary

Calling mutations in guide RNA (gRNA) sequences by comparing UMI–cell-barcode consensus sequences against the oligo pool plasmid reference, annotating mutation positions with a structure GTF, as implemented in GiRAFR's variant-calling step for single-cell CRISPR screens.

## When to use

Use when single-cell CRISPR screen gRNA alignments need mutation-aware calling — i.e., you have a consensus.bam of UMI-cell barcode consensus sequences and an oligo_pool_plasmid.fa reference, and must distinguish intact from mutant gRNAs before assigning gRNAs to cells.

## When NOT to use

- The analysis goal is CRISPR-Cas9 editing-effect detection in transcriptome data (deletions passing cutsite) rather than gRNA library mutation profiling — GiRAFR provides a separate analysis mode for that.
- Input is a gene-expression matrix or an already-computed gRNA-to-cell assignment table; variant calling is upstream of annotation and cannot recover mutation information from aggregated outputs.
- No plasmid oligo pool reference (oligo_pool_plasmid.fa / structure.gtf) is available, since consensus sequences must be compared to the designed gRNA reference to call variants.

## Inputs

- gRNA alignment bam file from the gRNA library sequencing (input to utils.gRNA_bam_filter)
- Filtered bam gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam
- Filtered cell barcode list
- consensus.bam containing consensus sequences for each UMI-cell barcode combination
- oligo_pool_plasmid.fa reference FASTA (from prepare.py / oligo_pool.csv with oligo_name and sequence columns)
- oligo_pool_plasmid_structure.gtf annotating mutation positions
- min_reads parameter (integer, default 1)

## Outputs

- Mutation calls per gRNA consensus sequence, with positions annotated against the oligo pool structure
- Intact vs mutant gRNA classification per UMI-cell barcode
- gRNA.umi.threshold.txt (auto UMI-threshold output from assign_gRNA)
- cells.gRNA.txt gRNA-to-cell assignments with variant details added by assign_gRNA.add_variant_type

## How to apply

First filter the gRNA alignment bam with GiRAFR's utils.gRNA_bam_filter, which uses samtools to remove secondary alignments and alignments not mapping to the designed gRNA cassette, yielding gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam. Next, build consensus sequences for each UMI-cell barcode combination via consensus_sequence.generate_consensus_sequence_gRNA using the filtered barcode list and min_reads (default 1; UMIs with fewer reads are filtered out). Then call mutations with variant.call_gRNA_variant, comparing each consensus sequence to the oligo_pool_plasmid.fa reference (generated by prepare.py together with oligo_pool_plasmid.gtf and oligo_pool_plasmid_structure.gtf) and annotating mutation positions with the structure GTF. Finally, feed the variant calls into assign_gRNA.assign_gRNA_to_cell (e.g., auto UMI-threshold mode with a two-component Gaussian mixture fit per gRNA, pooling intact and mutated UMIs), with assign_gRNA.add_variant_type adding mutation details for downstream analysis. Rationale: the default CellRanger-style approach labels gRNAs with a single mutation (1 Hamming distance) as intact, which attenuates perturbation phenotypes, so mutation-aware calling is required for accurate singlet/multiplet and untransformed-cell estimates.

## Related tools

- **GiRAFR** (Python/pysam package whose variant module (variant.call_gRNA_variant) performs the mutation calling against the oligo pool reference and structure GTF; also provides bam filtering, consensus construction, and gRNA-to-cell assignment) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **pysam** (Python wrapper of the samtools C-API used to read and manipulate the SAM/BAM gRNA alignments underlying variant calling) — https://github.com/pysam-developers/pysam
- **samtools** (Splits and filters the gRNA bam file, removing secondary alignments and off-cassette alignments before consensus construction and variant calling)
- **Python** (Implementation language for GiRAFR's variant-calling and annotation workflow) — https://github.com/FunctionalEpigeneticsLab/GiRAFR

## Examples

```
variant.call_gRNA_variant(consensus_seq_file, ref_fasta, structure_gtf)  # e.g. consensus.bam vs oligo_pool_plasmid.fa with oligo_pool_plasmid_structure.gtf
```

## Evaluation signals

- Filtered bam exists and is named gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam, confirming samtools-based filtration removed secondary and off-cassette alignments.
- Consensus sequences are produced for every UMI-cell barcode combination; UMIs with fewer than min_reads reads (default 1) are absent from the output.
- Each consensus sequence maps to exactly one oligo in oligo_pool_plasmid.fa, and called mutation positions are consistent with coordinates in oligo_pool_plasmid_structure.gtf.
- Downstream sanity: cells classified as mutant-only should reduce the inflated untransformed-cell estimate and raise the multiplet estimate relative to a CellRanger annotation that labels 1-Hamming-distance gRNAs as intact.
- In a reproduction test, annotation accuracy (~80%, ~95%, ~98%) and median UMIs per cell (~5, ~16, ~31) at 34%, 59%, and 92% saturation should match Figure 3b when the full GiRAFR pipeline including variant calling is applied to downsampled bams.

## Limitations

- Mutant gRNAs observed are reported to be dysfunctional, but which mutation types and locations are tolerable or damaging was not appraised — declared beyond the scope of the study.
- CellRanger-style pipelines label gRNAs with a single mutation (1 Hamming distance) as intact, so variant-calling results may conflict with pre-existing singlet annotations (up to 22% of CellRanger singlets are in fact multiplets; up to 34-35% of cells in public datasets carry a mutant gRNA).
- Specific defined read depths need to be attained for accurate gRNA assignment and mutation detection; insufficient sequencing saturation degrades both mutation calling and annotation accuracy.

## Evidence

- [methods] Step3: Call mutations from consensus.bam: "Step3: Call mutations from ``consensus.bam``"
- [methods] variant.call_gRNA_variant signature: "variant.call_gRNA_variant(consensus_seq_file, ref_fasta, structure_gtf)"
- [methods] Consensus construction per UMI-cell barcode combination: "Step2: Construct consensus sequence for each UMI-Cell barcode combination"
- [methods] min_reads filtering of low-support UMIs: "Integer number. By default = 1. UMIs with less than min_reads will be filtered out."
- [methods] prepare.py generates the plasmid reference and structure annotation used for variant calling: "prepare.py: generate ``oligo_pool_plasmid.fa``, ``oligo_pool_plasmid.gtf`` and ``oligo_pool_plasmid_structure.gtf``"
- [readme] GiRAFR is mutation-aware gRNA assignment software: "GiRAFR is a python package developed to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells in a sensitive, mutation-aware manner."
