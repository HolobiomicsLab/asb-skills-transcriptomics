---
name: gaussian-mixture-umi-threshold-fitting
description: Use when assigning gRNAs to cells in single-cell CRISPR screens whose
  per-gRNA UMI distributions vary widely (low vs high UMI experiments), where a fixed
  min_umi threshold over- or under-assigns cells; triggered by UMI-histogram bimodality
  or inter-experiment differences in average gRNA UMIs.
when_to_use_negative:
- When gRNA UMI distributions are shallow/uninformative — the article notes it is
  undetermined whether low average gRNA UMIs stem from low gRNA expression or low
  detection rates, which can detrimentally impact cell assignment accuracy regardless
  of thresholding
- When input is not a GiRAFR consensus/UMI table (e.g. already a cell-by-gRNA assignment
  matrix) — the mixture fits per-gRNA UMI count distributions produced upstream
- When studying CRISPR-Cas9 editing effects in the transcriptome rather than gRNA
  library annotation — GiRAFR handles that in a separate analysis mode
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: GiRAFR
  role: Python package providing assign_gRNA.assign_gRNA_to_cell with the auto two-component
    Gaussian mixture thresholding and pool option
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: pysam
  role: pysam-based backend on which GiRAFR reads/manipulates the gRNA BAM alignment
    data feeding assignment
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:80873c2455f72e43137a467075ef7d64f73ead893752f9364151047178a15f5c
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# gaussian-mixture-umi-threshold-fitting

## Summary

A gRNA-to-cell assignment strategy from GiRAFR that replaces the fixed min_umi cutoff (default = 3) with a per-gRNA UMI threshold obtained by fitting a two-component Gaussian mixture to the UMI count distribution, separating intact and mutant gRNA UMIs (pool=True by default).

## When to use

Use when assigning gRNAs to cells in single-cell CRISPR screens whose per-gRNA UMI distributions vary widely (low vs high UMI experiments), where a fixed min_umi threshold over- or under-assigns cells; triggered by UMI-histogram bimodality or inter-experiment differences in average gRNA UMIs.

## When NOT to use

- When gRNA UMI distributions are shallow/uninformative — the article notes it is undetermined whether low average gRNA UMIs stem from low gRNA expression or low detection rates, which can detrimentally impact cell assignment accuracy regardless of thresholding
- When input is not a GiRAFR consensus/UMI table (e.g. already a cell-by-gRNA assignment matrix) — the mixture fits per-gRNA UMI count distributions produced upstream
- When studying CRISPR-Cas9 editing effects in the transcriptome rather than gRNA library annotation — GiRAFR handles that in a separate analysis mode

## Inputs

- Consensus sequence / UMI count table per UMI-cell barcode combination (output of consensus_sequence.generate_consensus_sequence_gRNA with min_reads filtering)
- gRNA variant calls with CIGAR-like mutation annotations from variant.call_gRNA_variant
- Parameters: min_umi (default = 3) when auto=False, auto=True, pool=True

## Outputs

- cells.gRNA.txt (cell-to-gRNA assignments)
- cells.gRNA.single.txt (single gRNA assignment per cell)
- gRNA.umi.threshold.txt (per-gRNA fitted UMI thresholds)
- Post-assignment mutation annotations via assign_gRNA.add_variant_type and profile_MT_pattern.py

## How to apply

After consensus sequence construction and mutation calling (variant.call_gRNA_variant against the gRNA cassette ref_fasta and structure_gtf), run assign_gRNA.assign_gRNA_to_cell on the consensus/UMI table. With auto=True, fit a two-component Gaussian mixture to the UMI counts of each gRNA to derive a data-driven UMI threshold instead of the default min_umi = 3 (cells with UMIs >= threshold are assigned the gRNA); by default pool=True so intact and mutated UMI counts are pooled when fitting, but pooling can be disabled to threshold each class separately. Thresholds are written to gRNA.umi.threshold.txt, and cell-gRNA assignments to cells.gRNA.txt (with a single-assignment variant cells.gRNA.single.txt). Rationale: fixed thresholds are inappropriate because average gRNA UMI counts differ dramatically between experiments, which harms accuracy of cell assignment and singlet vs multiplet discrimination.

## Related tools

- **GiRAFR** (Python package providing assign_gRNA.assign_gRNA_to_cell with the auto two-component Gaussian mixture thresholding and pool option) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **pysam** (pysam-based backend on which GiRAFR reads/manipulates the gRNA BAM alignment data feeding assignment) — https://github.com/pysam-developers/pysam

## Examples

```
from girafr import assign_gRNA; assign_gRNA.assign_gRNA_to_cell(auto=True, pool=True, min_umi=3)
```

## Evaluation signals

- gRNA.umi.threshold.txt contains a fitted threshold for each gRNA, and cells.gRNA.txt exists and matches the expected GiRAFR output format
- Per-experiment mutant gRNA frequencies fall within the reported range (2%–35% of cells; up to 35% in public datasets)
- Cells assigned with UMIs >= the per-gRNA threshold; setting auto=False with min_umi=3 reproduces the fixed-threshold baseline for comparison
- Per-experiment mutation rates per 1000 nucleotides tabulated for the 13 high-depth public experiments are consistent with Figure 2d-e values
- Singlet/multiplet and untransformed-cell counts shift as expected: mutation-aware assignment reduces the inflated untransformed-cell estimate and corrects the underestimated multiplet frequency (up to 22% of Cell Ranger 'singlets' are actually multiplets)

## Limitations

- Head-to-head comparison between published studies is unfeasible because studies use ad hoc customized pipelines and fixed thresholds
- Specific defined read depths need to be attained for accurate gRNA assignment and mutation detection
- Dramatic inter-experiment differences in average gRNA UMIs (low gRNA expression vs low detection rates) undetermined, impacting assignment accuracy, ambient vs endogenous discrimination, and singlet vs multiplet discrimination
- Design choices (MOI, number of designed gRNAs, introduced mutations, cells per perturbation) influence assignment accuracy and require parameter finetuning

## Evidence

- [methods] Auto UMI threshold via Gaussian mixture: [evidence span withheld — anchor status: not_found]
- [methods] Pool intact and mutated UMIs: [evidence span withheld — anchor status: not_found]
- [discussion] Fixed threshold inadequacy: "studies often use ad hoc customized pipelines and fixed thresholds for analysis. This renders a head-to-head comparison between each of these studies unfeasible."
- [discussion] UMI variability impacts assignment: "This may either be due to low gRNA expression or low detection rates, but can have a detrimental impact on the accuracy of assigning gRNAs to cells"
- [readme] README output scope: "GiRAFR is a python package developed to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells in a sensitive, mutation-aware manner."
