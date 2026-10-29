---
name: per-grna-umi-threshold-estimation
description: Use when assigning CRISPR gRNAs to cells from UMI counts per cell barcode,
  when fixed UMI thresholds misclassify cells due to varying gRNA expression levels,
  or when discriminating ambient from endogenously expressed gRNAs.
when_to_use_negative:
- Input is already a cell-by-gRNA assignment matrix or feature table rather than raw
  per-UMI consensus sequences needing thresholding
- The study requires comparing gRNA assignment across experiments that used ad hoc
  customized pipelines with fixed thresholds — the authors note this renders head-to-head
  comparison unfeasible
- The goal is fine-grained appraisal of which gRNA mutation types or locations are
  tolerable or damaging, which the authors declare beyond the scope of this assignment
  step
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: GiRAFR
  role: Python package whose assign_gRNA.assign_gRNA_to_cell module performs Gaussian-mixture-based
    per-gRNA UMI thresholding and cell assignment
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: Python
  role: Implementation language for GiRAFR and the scikit-learn Gaussian mixture fitting
- name: sklearn
  role: Provides the two-component GaussianMixture model fitted per gRNA to separate
    endogenous from ambient UMI distributions
- name: Cell Ranger
  role: Reference implementation whose two-component Gaussian mixture thresholding
    approach GiRAFR adopts for dynamic UMI thresholds
- name: pysam
  role: BAM-file backend underlying GiRAFR's upstream consensus generation feeding
    the assignment step
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:7f1007678f23a936fb21ba674578c8706cdbb9de3cad33b04bb656327c8879bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FunctionalEpigeneticsLab/GiRAFR
  tools:
  - Cell Ranger
  - GiRAFR
  - Python
  - pysam
  - sklearn
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# per-grna-umi-threshold-estimation

## Summary

Assigns gRNAs to single cells by estimating dynamic per-gRNA UMI thresholds with a two-component Gaussian mixture model (scikit-learn), replacing the fixed min_umi=3 cutoff, and distinguishing singlets from multiplets in single-cell CRISPR screen data.

## When to use

Use when assigning CRISPR gRNAs to cells from UMI counts per cell barcode, when fixed UMI thresholds misclassify cells due to varying gRNA expression levels, or when discriminating ambient from endogenously expressed gRNAs.

## When NOT to use

- Input is already a cell-by-gRNA assignment matrix or feature table rather than raw per-UMI consensus sequences needing thresholding
- The study requires comparing gRNA assignment across experiments that used ad hoc customized pipelines with fixed thresholds — the authors note this renders head-to-head comparison unfeasible
- The goal is fine-grained appraisal of which gRNA mutation types or locations are tolerable or damaging, which the authors declare beyond the scope of this assignment step

## Inputs

- consensus.sequence.gRNA.variant.txt (per-UMI mutation calls from consensus.bam)
- Filtered cell barcode list from the expression library mapping
- Per-gRNA per-cell UMI counts (intact and mutant pooled)

## Outputs

- gRNA.umi.threshold.txt (one fitted threshold per gRNA)
- consensus.sequence.matrix
- cells.gRNA.txt (cell-to-gRNA assignments)
- cells.gRNA.single.txt (singlet assignments)

## How to apply

Load per-UMI mutation calls from consensus.sequence.gRNA.variant.txt together with the filtered cell barcode list produced by the expression library mapping. For each gRNA, pool intact and mutant UMI counts together (pool=True) so a single distribution per guide is modeled, since mutant gRNAs are common and must be captured in the assignment. Fit a two-component Gaussian mixture model per gRNA with scikit-learn (auto=True) — the same approach used by Cell Ranger — to separate endogenous in-cell UMIs from ambient UMIs and derive a per-guide threshold automatically; fall back to the fixed min_umi=3 default when auto is disabled. Assign gRNAs to cells whose UMI counts meet the fitted threshold, distinguishing singlets from multiplets, and write gRNA.umi.threshold.txt, consensus.sequence.matrix, cells.gRNA.txt, and cells.gRNA.single.txt.

## Related tools

- **GiRAFR** (Python package whose assign_gRNA.assign_gRNA_to_cell module performs Gaussian-mixture-based per-gRNA UMI thresholding and cell assignment) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **Python** (Implementation language for GiRAFR and the scikit-learn Gaussian mixture fitting)
- **sklearn** (Provides the two-component GaussianMixture model fitted per gRNA to separate endogenous from ambient UMI distributions)
- **Cell Ranger** (Reference implementation whose two-component Gaussian mixture thresholding approach GiRAFR adopts for dynamic UMI thresholds)
- **pysam** (BAM-file backend underlying GiRAFR's upstream consensus generation feeding the assignment step) — https://github.com/pysam-developers/pysam

## Examples

```
from girafr import assign_gRNA; assign_gRNA.assign_gRNA_to_cell()
```

## Evaluation signals

- cells.gRNA.txt and cells.gRNA.single.txt exist and are populated only with cells whose gRNA UMI counts satisfy the fitted threshold
- gRNA.umi.threshold.txt contains exactly one threshold per gRNA
- With auto=True, thresholds vary per gRNA rather than all equaling min_umi=3
- Singlet vs multiplet calls in cells.gRNA.single.txt are consistent with the number of gRNAs passing threshold per cell in cells.gRNA.txt
- Multiplet frequency should be lower-biased relative to Cell Ranger assignments, since GiRAFR captures mutant gRNAs that Cell Ranger labels as intact

## Limitations

- Inter-experiment differences in average gRNA UMIs — whether from low gRNA expression or low detection rates — can detrimentally impact accuracy of assigning gRNAs to cells and discriminating ambient from endogenously expressed gRNAs
- Design choices (multiplicity of infection, number of designed gRNAs, introduced mutations, cells per perturbation) influence assignment accuracy and require optimization or parameter finetuning
- Specific defined read depths need to be attained for accurate gRNA assignment
- The two-component mixture models both mutant and intact UMI counts together (pool=True), which assumes a single distribution per guide adequately separates ambient from endogenous signal

## Evidence

- [methods] two thresholding modes: [evidence span withheld — anchor status: not_found]
- [methods] gmm per grna pooling intact and mutant: [evidence span withheld — anchor status: not_found]
- [methods] min_umi default: "Integer number. By default = 3. Cells with UMIs >= min_umi will be assigned with gRNA."
- [intro] assign step purpose: "GiRAFR ... to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells in a sensitive, mutation-aware manner."
- [discussion] ambient vs endogenous sensitivity: "This may either be due to low gRNA expression or low detection rates, but can have a detrimental impact on the accuracy of assigning gRNAs to cells, discriminating ambient from endogenously expressed"
