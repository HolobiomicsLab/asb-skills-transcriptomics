---
name: grna-library-sequencing-downsampling
description: Use when estimating the minimum gRNA library sequencing depth needed
  for accurate gRNA-to-cell assignment in a CROP-seq experiment, by comparing annotation
  accuracy and UMIs per cell across downsampled saturation levels.
when_to_use_negative:
- The gRNA library is already deeply saturated and the goal is mutation-aware annotation
  only — run the GiRAFR gRNA_mutation pipeline directly without downsampling.
- Analyzing the transcriptome side of the CROP-seq experiment (gene expression matrices,
  perturbation phenotypes) rather than gRNA library depth and assignment accuracy.
- Estimating CRISPR-Cas9 editing effects from transcriptome data — use GiRAFR's separate
  editing-effect analysis mode instead.
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3173
tools:
- name: GiRAFR
  role: gRNA_mutation pipeline (gRNA_bam_filter, consensus_sequence, variant calling,
    assign_gRNA_to_cell) rerun per downsampled BAM; pip-installable python package
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: samtools
  role: random subsampling of the gRNA library BAM to target saturation levels and
    splitting/filtering alignment files in step 1 of gRNA_mutation
- name: pysam
  role: pysam-based backend that GiRAFR is built on for reading and manipulating the
    SAM/BAM gRNA alignments
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:ebe7fc168c2f2a00fa3be1dda50287825543312a2d773bc6b6a7a12621ab0c5c
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
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# grna-library-sequencing-downsampling

## Summary

Assesses gRNA library sequencing saturation by downsampling the gRNA alignment BAM (e.g. 34%, 59%, 92%) and rerunning the GiRAFR gRNA_mutation pipeline at each depth to quantify how saturation affects gRNA-to-cell annotation accuracy and unique gRNA UMI recovery per cell.

## When to use

Use when estimating the minimum gRNA library sequencing depth needed for accurate gRNA-to-cell assignment in a CROP-seq experiment, by comparing annotation accuracy and UMIs per cell across downsampled saturation levels.

## When NOT to use

- The gRNA library is already deeply saturated and the goal is mutation-aware annotation only — run the GiRAFR gRNA_mutation pipeline directly without downsampling.
- Analyzing the transcriptome side of the CROP-seq experiment (gene expression matrices, perturbation phenotypes) rather than gRNA library depth and assignment accuracy.
- Estimating CRISPR-Cas9 editing effects from transcriptome data — use GiRAFR's separate editing-effect analysis mode instead.

## Inputs

- gRNA library alignment BAM file (original, fully saturated sequencing)
- oligo_pool_plasmid.fa reference of designed gRNA sequences
- oligo_pool_plasmid_structure.gtf (gRNA structure annotation for mutation positions)
- filtered cell barcode list

## Outputs

- one downsampled BAM per saturation level (e.g. 34%, 59%, 92%)
- gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam (filtered BAM per level)
- consensus.bam (UMI-cell barcode consensus sequences per level)
- cells.gRNA.txt and gRNA.umi.threshold.txt per level
- saturation_annotation_metrics.csv (annotation accuracy and median UMIs per cell at each saturation)

## How to apply

Randomly subsample the gRNA library alignment BAM to target saturation levels (e.g. 34%, 59%, 92% of original reads) with samtools view random subsampling, producing one downsampled BAM per level. For each downsampled BAM, run the GiRAFR gRNA_mutation pipeline: gRNA bam filtration (samtools-based removal of secondary alignments and alignments not mapping to the designed gRNA cassette) yielding gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam, then consensus_sequence.generate_consensus_sequence_gRNA on the filtered barcode list with min_reads=1 to build UMI-cell barcode consensus sequences. Call mutations with variant.call_gRNA_variant against oligo_pool_plasmid.fa using oligo_pool_plasmid_structure.gtf for mutation position annotation, then assign gRNAs to cells with assign_gRNA.assign_gRNA_to_cell in auto UMI-threshold mode (two-component Gaussian mixture fit per gRNA over pooled intact and mutated UMIs). At each saturation, compute gRNA annotation accuracy and median UMIs per cell from cells.gRNA.txt and gRNA.umi.threshold.txt and compile them into saturation_annotation_metrics.csv; accuracy should rise with saturation as more unique gRNA molecules per cell are recovered.

## Related tools

- **GiRAFR** (gRNA_mutation pipeline (gRNA_bam_filter, consensus_sequence, variant calling, assign_gRNA_to_cell) rerun per downsampled BAM; pip-installable python package) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **samtools** (random subsampling of the gRNA library BAM to target saturation levels and splitting/filtering alignment files in step 1 of gRNA_mutation)
- **pysam** (pysam-based backend that GiRAFR is built on for reading and manipulating the SAM/BAM gRNA alignments) — https://github.com/pysam-developers/pysam

## Examples

```
samtools view -s 0.59 -o downsampled_59pct.bam gRNA_library.bam; python -c "from girafr import assign_gRNA; assign_gRNA.assign_gRNA_to_cell()"
```

## Evaluation signals

- Reproduced annotation accuracies at 34%, 59%, and 92% saturation should be approximately 80%, 95%, and 98%, matching Figure 3b of the paper.
- Median/average unique gRNA molecules recovered per cell should be approximately 5, 16, and 31 at 34%, 59%, and 92% saturation respectively.
- Annotation accuracy and UMIs per cell must increase monotonically with sequencing saturation across the three downsampled levels.
- gRNA.umi.threshold.txt shows a per-gRNA UMI threshold fitted as a two-component Gaussian mixture over pooled intact and mutated UMIs (auto mode) at every saturation level.
- saturation_annotation_metrics.csv contains exactly one row per saturation level with accuracy and median UMIs per cell computed from cells.gRNA.txt and gRNA.umi.threshold.txt.

## Limitations

- Specific defined read depths need to be attained for accurate gRNA assignment and mutation detection; the accuracy/saturation curve must be established per library rather than assumed.
- Inter-experiment differences in average gRNA UMIs may stem from low gRNA expression or low detection rates, which the downsampling curve alone does not disentangle.
- Downsampling yields accuracy for the in-house experiment's library design; design choices such as multiplicity of infection, number of designed gRNAs, and mutations introduced in them influence assignment accuracy and require per-experiment optimization.

## Evidence

- [readme] GiRAFR is a python package developed to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells in a sensitive, mutation-aware manner.: "GiRAFR is a python package developed to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells"
- [methods] step 1: gRNA bam file filtration.: "step 1: gRNA bam file filtration."
- [methods] Step2: Construct consensus sequence for each UMI-Cell barcode combination: "Step2: Construct consensus sequence for each UMI-Cell barcode combination"
- [methods] Integer number. By default = 1. UMIs with less than min_reads will be filtered out.: "Integer number. By default = 1. UMIs with less than min_reads will be filtered out."
- [methods] assign_gRNA.assign_gRNA_to_cell(): "assign_gRNA.assign_gRNA_to_cell()"
