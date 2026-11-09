---
name: single-cell-grna-assignment-threshold-tuning
description: Use when assigning gRNAs to cells from a Perturb-seq/CROP-seq gRNA capture
  library and you need mutation-aware cell-to-guide assignment with adaptive UMI thresholds
  rather than fixed ad hoc cutoffs.
when_to_use_negative:
- You only need to detect CRISPR-Cas9-induced DNA editing effects in the transcriptome
  — use GiRAFR's separate editing-effect mode instead of cell assignment
- Cell-to-guide assignments already exist from a trusted, validated annotation and
  you are not re-profiling gRNA sequences or mutations
- The gRNA library has not been aligned to a custom cassette reference, so no consensus
  sequences or variant calls exist to feed assignment
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: GiRAFR
  role: Provides assign_gRNA.assign_gRNA_to_cell, which fits per-guide UMI thresholds
    and assigns gRNAs (intact and mutant) to cells
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: pysam
  role: Underlying python library GiRAFR is built on for reading the gRNA BAM/consensus
    alignments
  repo: https://github.com/pysam-developers/pysam
- name: samtools
  role: Used upstream by utils.gRNA_bam_filter to produce the filtered cassette-only
    BAM that assignment consumes
- name: CellRanger
  role: Supplies the filtered cell barcode list from the expression library; its default
    gRNA handling is mutation-unaware and is what GiRAFR corrects
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:0fe988061351b120932cc6711843763895efdfe04969435736201f5856a17cef
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
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# single-cell-grna-assignment-threshold-tuning

## Summary

Assign gRNAs to individual cells in single-cell CRISPR screens using a data-adaptive, per-guide UMI threshold fitted with a two-component mixed Gaussian model, so that low-UMI ambient signal is separated from endogenous gRNA expression and mutant gRNAs are treated on par with intact ones.

## When to use

Use when assigning gRNAs to cells from a Perturb-seq/CROP-seq gRNA capture library and you need mutation-aware cell-to-guide assignment with adaptive UMI thresholds rather than fixed ad hoc cutoffs.

## When NOT to use

- You only need to detect CRISPR-Cas9-induced DNA editing effects in the transcriptome — use GiRAFR's separate editing-effect mode instead of cell assignment
- Cell-to-guide assignments already exist from a trusted, validated annotation and you are not re-profiling gRNA sequences or mutations
- The gRNA library has not been aligned to a custom cassette reference, so no consensus sequences or variant calls exist to feed assignment

## Inputs

- consensus.sequence.gRNA.variant.txt (consensus gRNA sequences with called mutations and CIGAR-like position annotations)
- consensus.sequence.gRNA.txt / consensus.sequence.matrix (per-UMI-cell-barcode consensus sequences)
- Filtered (unzipped) cell barcode list from the expression library (e.g., Cell Ranger filtered barcodes)
- gRNA library alignment BAM filtered to cassette chromosomes with secondary alignments removed (gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam)
- Reference gRNA cassette fasta (oligo_pool_plasmid.fa) and structure gtf (oligo_pool_plasmid_structure.gtf)

## Outputs

- cells.gRNA.txt (gRNA assignments per cell, including mutant guides)
- cells.gRNA.single.txt (singlet assignments only)
- gRNA.umi.threshold.txt (per-guide fitted UMI thresholds from the mixed Gaussian model)
- consensus.sequence.matrix (consensus sequence count matrix)
- Optional: variant-type annotated tables from assign_gRNA.add_variant_type / profile_MT_pattern.py

## How to apply

After variant calling produces consensus.sequence.gRNA.variant.txt from the UMI-cell barcode consensus BAM, run assign_gRNA.assign_gRNA_to_cell with min_umi=3 so only cells carrying at least 3 gRNA UMIs are assigned. Set auto=True to fit a two-component mixed Gaussian model per guide and derive data-driven UMI thresholds (written to gRNA.umi.threshold.txt) instead of imposing fixed cutoffs, which the paper notes makes studies incomparable. Use pool=True so intact and mutated UMIs of the same guide are pooled for threshold fitting, ensuring mutant gRNAs — which GiRAFR recovers alongside intact ones — receive sensible thresholds. The function writes cells.gRNA.txt (all assignments), cells.gRNA.single.txt (singlets), and consensus.sequence.matrix; optionally run assign_gRNA.add_variant_type and profile_MT_pattern.py to attach mutation details for downstream analysis. Validate by comparing assignments against deposited cell annotations (e.g., 98.1% concordance among 14,113 mutant-gRNA-annotated cells) and by checking that threshold files contain one fitted threshold per guide.

## Related tools

- **GiRAFR** (Provides assign_gRNA.assign_gRNA_to_cell, which fits per-guide UMI thresholds and assigns gRNAs (intact and mutant) to cells) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **pysam** (Underlying python library GiRAFR is built on for reading the gRNA BAM/consensus alignments) — https://github.com/pysam-developers/pysam
- **samtools** (Used upstream by utils.gRNA_bam_filter to produce the filtered cassette-only BAM that assignment consumes)
- **CellRanger** (Supplies the filtered cell barcode list from the expression library; its default gRNA handling is mutation-unaware and is what GiRAFR corrects)

## Examples

```
assign_gRNA.assign_gRNA_to_cell(min_umi=3, auto=True, pool=True)  # inputs: consensus.sequence.gRNA.variant.txt, filtered barcodes; outputs: cells.gRNA.txt, cells.gRNA.single.txt, gRNA.umi.threshold.txt
```

## Evaluation signals

- All 103 predesigned mutant gRNAs recovered and ≥98.1% concordant cell assignment among the 14,113 originally annotated mutant-gRNA cells (as in the reproduced Perturb-seq benchmark)
- gRNA.umi.threshold.txt contains a fitted threshold for every guide in the library (auto=True two-component mixed Gaussian converged per guide)
- No cell in cells.gRNA.txt has fewer than min_umi=3 gRNA UMIs assigned
- Singlets in cells.gRNA.single.txt are consistent subset of cells.gRNA.txt; multiplet rate is plausible (paper notes Cell Ranger singlets can hide up to 22% multiplets)
- Assignment counts are not inflated by mutant-gRNA loss: number of 'untransformed' cells does not grow relative to a mutation-aware baseline

## Limitations

- Accurate assignment requires adequate read depth: 'specific defined read depths need to be attained for accurate gRNA assignment and mutation detection'
- Inter-experiment differences in average gRNA UMIs (low expression vs low detection) can hurt assignment accuracy and ambient-vs-endogenous discrimination; the paper does not resolve how to distinguish these
- Design choices (multiplicity of infection, number of designed gRNAs, mutations present, cells per perturbation) influence accuracy and may require parameter finetuning beyond defaults
- Fixed or ad hoc thresholds across studies render head-to-head comparisons unfeasible; thresholds here are dataset-specific and not transferable

## Evidence

- [methods] min_umi threshold: "Integer number. By default = 3. Cells with UMIs >= min_umi will be assigned with gRNA."
- [methods] min_reads threshold: "Integer number. By default = 1. UMIs with less than min_reads will be filtered out."
- [methods] assignment step: "Step4: Assign gRNAs to cells"
- [discussion] mutation-aware rationale: "we developed a method to identify and annotate the gRNAs used in these assays more accurately, in a mutation-aware manner. GiRAFR takes alignment results (bam files) as its main input."
- [discussion] multiplet correction motivation: "up to 22% of all cells profiled in earlier studies that are labeled as singlets by Cell Ranger are in fact multiplets"
- [readme] tool purpose from README: "GiRAFR is a python package developed to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells in a sensitive, mutation-aware manner."
- [discussion] adaptive thresholding rationale: "studies often use ad hoc customized pipelines and fixed thresholds for analysis. This renders a head-to-head comparison between each of these studies unfeasible."
