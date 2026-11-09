---
name: single-cell-crispr-screen-quality-control
description: Use when a single-cell CRISPR screen (e.g. CROP-seq) gRNA library BAM
  must be quality-controlled for mutant gRNAs and cells must be assigned gRNAs mutation-aware,
  beyond Cell Ranger feature barcoding.
when_to_use_negative:
- When only intact-gRNA assignment via the standard Cell Ranger feature barcoding
  pipeline is sufficient and mutation profiling is not the goal (Cell Ranger misses
  mutant gRNAs and labels 1-Hamming-distance gRNAs as intact).
- For the separate GiRAFR analysis mode that detects CRISPR-Cas9-induced DNA editing
  in transcriptome data — that is a different mode from gRNA library mutation profiling.
- For head-to-head comparison across published studies when their ad hoc customized
  pipelines and fixed thresholds are incompatible; the article notes such comparisons
  are unfeasible.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: GiRAFR
  role: Core pysam-based python package performing BAM filtration, UMI-cell consensus
    building, mutation calling, and gRNA-to-cell assignment
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: pysam
  role: Underlying library GiRAFR is built on for reading/manipulating SAM/BAM alignments
  repo: https://github.com/pysam-developers/pysam
- name: samtools
  role: Used to split and filter the gRNA alignment BAM during the bam filtration
    step
- name: CellRanger
  role: Provides corrected cell barcodes (filtered_barcode input) and the baseline
    annotation GiRAFR assignments are compared against
- name: umi_tools
  role: FeatureCount from UMI-tools is employed to filter out reads that do not align
    properly to the target gene
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:08fb1a75b416ac39b525553b46c18a0767168a32757849fa1ba4d296f2648928
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
  - umi_tools
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# single-cell-crispr-screen-quality-control

## Summary

A mutation-aware quality-control workflow for single-cell CRISPR screens that profiles gRNA sequence variations, quantifies mutant gRNA molecules in the library and in cells, and assigns intact or mutated gRNAs to cells using GiRAFR rather than the default Cell Ranger feature barcoding.

## When to use

Use when a single-cell CRISPR screen (e.g. CROP-seq) gRNA library BAM must be quality-controlled for mutant gRNAs and cells must be assigned gRNAs mutation-aware, beyond Cell Ranger feature barcoding.

## When NOT to use

- When only intact-gRNA assignment via the standard Cell Ranger feature barcoding pipeline is sufficient and mutation profiling is not the goal (Cell Ranger misses mutant gRNAs and labels 1-Hamming-distance gRNAs as intact).
- For the separate GiRAFR analysis mode that detects CRISPR-Cas9-induced DNA editing in transcriptome data — that is a different mode from gRNA library mutation profiling.
- For head-to-head comparison across published studies when their ad hoc customized pipelines and fixed thresholds are incompatible; the article notes such comparisons are unfeasible.

## Inputs

- gRNA library alignment BAM (mapped to the custom gRNA-cassette reference)
- Filtered/unzipped Cell Ranger corrected cell barcodes list
- Reference gRNA cassette FASTA (e.g. oligo_pool_plasmid.fa)
- Structure annotation GTF (e.g. oligo_pool_plasmid_structure.gtf)
- GiRAFR ConfigFile specifying gRNA_bam_file, filtered_barcode, ref_fasta, structure_gtf

## Outputs

- gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam (filtered BAM)
- consensus.sequence.gRNA.txt (per-UMI consensus sequences)
- consensus.sequence.gRNA.variant.txt (variants in CIGAR-like string format)
- cells.gRNA.txt, cells.gRNA.single.txt, gRNA.umi.threshold.txt (per-cell gRNA assignments and thresholds)

## How to apply

Build a custom reference from the oligo pool (oligo_pool_plasmid.fa plus oligo_pool_plasmid_structure.gtf via prepare.py), then run the four-step GiRAFR gRNA_mutation pipeline launched with 'girafr gRNA_mutation -f ConfigFile'. Step 1: filter the gRNA library BAM with utils.gRNA_bam_filter and samtools to remove secondary alignments and reads not mapped to the designed gRNA cassette, yielding gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam. Step 2: construct a consensus sequence per UMI-cell barcode combination with consensus_sequence.generate_consensus_sequence_gRNA (min_reads default = 1; use min_reads = 2 to discard UMIs attributable to sequencing errors). Step 3: call mutations with variant.call_gRNA_variant against the reference FASTA with structure-GTF annotation, writing variants as CIGAR-like strings to consensus.sequence.gRNA.variant.txt. Step 4: assign gRNAs to cells with assign_gRNA.assign_gRNA_to_cell using min_umi = 3 as default threshold or auto = True two-component Gaussian mixture per-gRNA UMI thresholding (pool = True pools intact and mutated UMIs), then annotate mutation types and locations downstream with assign_gRNA.add_variant_type and profile_MT_pattern.py.

## Related tools

- **GiRAFR** (Core pysam-based python package performing BAM filtration, UMI-cell consensus building, mutation calling, and gRNA-to-cell assignment) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **pysam** (Underlying library GiRAFR is built on for reading/manipulating SAM/BAM alignments) — https://github.com/pysam-developers/pysam
- **samtools** (Used to split and filter the gRNA alignment BAM during the bam filtration step)
- **CellRanger** (Provides corrected cell barcodes (filtered_barcode input) and the baseline annotation GiRAFR assignments are compared against)
- **umi_tools** (FeatureCount from UMI-tools is employed to filter out reads that do not align properly to the target gene)

## Examples

```
girafr gRNA_mutation -f ConfigFile
```

## Evaluation signals

- cells.gRNA.txt exists and matches the expected GiRAFR output format.
- Per-experiment mutant gRNA frequencies fall within the reported range (up to ~35% of cells transduced with a mutant gRNA in public datasets).
- On the in-house A549 CROP-seq data, GiRAFR assigns 1198/5744 (20.7%) cells a mutant gRNA, including 481 cells with no Cell Ranger gRNA call, and Cell Ranger leaves 2317/5744 cells without a gRNA call.
- With min_reads = 2 consensus filtering, the mutant gRNA molecule fraction in the in-house library matches the reported 28.4%.
- Downsampling checks: annotation accuracy ~80%, ~95%, ~98% and median UMIs per cell ~5, ~16, ~31 at 34%, 59%, 92% saturation (Figure 3b values).

## Limitations

- Fine-grained appraisal of which gRNA mutation types and locations are tolerable or damaging was not performed and is beyond the scope of the study.
- Accurate gRNA assignment and mutation detection require specific defined read depths; at low sequencing saturation accuracy drops (e.g. only ~80% at 34% saturation).
- Dramatic inter-experiment differences in average gRNA UMIs may stem from low gRNA expression or low detection rates — undetermined, affecting cell assignment, ambient-vs-endogenous and singlet-vs-multiplet discrimination.
- Design choices (multiplicity of infection, number of designed gRNAs, introduced mutations, cells per perturbation) influence assignment accuracy and require optimization or parameter finetuning.

## Evidence

- [intro] GiRAFR is a pysam-based tool for mutation-aware gRNA assignment in single-cell CRISPR screens: "GiRAFR (Guide RNA Anomaly and Functionality Revealer), a tool based on pysam"
- [methods] The gRNA_mutation pipeline has four named steps: BAM filtration, UMI-cell consensus, mutation calling, cell assignment: "Step4: Assign gRNAs to cells"
- [methods] min_reads default of 1 filters UMIs with too few reads; min_umi default of 3 gates cell assignment: "Integer number. By default = 3. Cells with UMIs >= min_umi will be assigned with gRNA."
- [abstract] Failure to detect mutant gRNAs inflates untransformed-cell estimates and underestimates multiplets: "failure to detect and annotate them leads to an inflated estimate of the number of untransformed cells, as well as an underestimated multiplet frequency"
- [abstract] Up to 35% of cells in public datasets carry a mutant gRNA: "These findings are mirrored in publicly available datasets, where we find that up to 35% of cells are transduced with a mutant gRNA."
- [readme] GiRAFR package performs QC of single-cell CRISPR screens and mutation-aware gRNA assignment: "GiRAFR is a python package developed to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells in a sensitive, mutation-aware manner."
