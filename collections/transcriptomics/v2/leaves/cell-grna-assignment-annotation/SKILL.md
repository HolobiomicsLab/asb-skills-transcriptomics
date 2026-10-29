---
name: cell-grna-assignment-annotation
description: Use when a single-cell CRISPR screen needs gRNAs assigned to cell barcodes
  from per-cell UMI counts, especially to separate ambient from endogenously expressed
  guides and distinguish singlets from multiplets.
when_to_use_negative:
- Input reads are not yet UMI-collapsed and consensus-called (consensus.sequence.gRNA.variant.txt
  missing) — run the earlier GiRAFR steps (gRNA bam filtration, consensus sequence
  construction, variant calling) first.
- The goal is detecting CRISPR-Cas9-induced DNA editing effects in transcriptome data,
  which GiRAFR handles in a separate analysis mode rather than via cell assignment.
- Only mutation patterns in the gRNA library itself need profiling (e.g. profile_MT_pattern.py
  outputs) without assigning guides to cells.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3391
tools:
- name: GiRAFR
  role: Python package whose assign_gRNA.assign_gRNA_to_cell() module performs the
    thresholding and cell assignment
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: sklearn
  role: Fits the two-component Gaussian mixture model per gRNA to derive dynamic UMI
    thresholds (auto=True)
- name: Cell Ranger
  role: Baseline tool whose fixed-threshold, mutation-blind assignment underestimates
    multiplets and labels mutant guides as intact; GiRAFR adopts its GMM strategy
    but is mutation-aware
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:4c87e74170f45c52e9a22836be5e91c2d3f98bd53ff07b28d06763b1ae9016fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FunctionalEpigeneticsLab/GiRAFR
  tools:
  - Cell Ranger
  - GiRAFR
  - Python
  - sklearn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# cell-grna-assignment-annotation

## Summary

Assign CRISPR guide RNAs to single cells from per-cell gRNA UMI counts in a mutation-aware manner, using either a fixed UMI cutoff (min_umi=3) or a two-component Gaussian mixture model to derive a dynamic threshold per gRNA. This is Step 4 of the GiRAFR pipeline and corrects the ambient-UMI-driven misannotation and underestimated multiplet rates seen with default tools.

## When to use

Use when a single-cell CRISPR screen needs gRNAs assigned to cell barcodes from per-cell UMI counts, especially to separate ambient from endogenously expressed guides and distinguish singlets from multiplets.

## When NOT to use

- Input reads are not yet UMI-collapsed and consensus-called (consensus.sequence.gRNA.variant.txt missing) — run the earlier GiRAFR steps (gRNA bam filtration, consensus sequence construction, variant calling) first.
- The goal is detecting CRISPR-Cas9-induced DNA editing effects in transcriptome data, which GiRAFR handles in a separate analysis mode rather than via cell assignment.
- Only mutation patterns in the gRNA library itself need profiling (e.g. profile_MT_pattern.py outputs) without assigning guides to cells.

## Inputs

- consensus.sequence.gRNA.variant.txt (per-UMI mutation calls from upstream GiRAFR steps)
- filtered cell barcode list from the expression library mapping
- per-cell intact and mutant gRNA UMI counts
- configuration parameters: min_umi (default 3), auto (GMM-based thresholding, default True), pool (default True)

## Outputs

- gRNA.umi.threshold.txt (one fitted UMI threshold per gRNA)
- consensus.sequence.matrix (gRNA-by-cell UMI/count matrix)
- cells.gRNA.txt (all gRNA-to-cell assignments meeting thresholds)
- cells.gRNA.single.txt (singlet assignments only)

## How to apply

Load the per-UMI mutation calls from consensus.sequence.gRNA.variant.txt together with the filtered cell barcode list produced by the expression library mapping. For each gRNA, pool intact and mutant UMI counts (pool=True) so a single distribution per guide is modeled — including mutant guides is essential because mutant gRNAs are dysfunctional and excluding them inflates the untransformed-cell estimate and underestimates multiplet frequency. Fit a two-component Gaussian mixture model per gRNA with scikit-learn (auto=True), the same strategy Cell Ranger uses, to derive a dynamic UMI threshold separating endogenous in-cell UMIs from ambient UMIs; when auto is disabled, fall back to the fixed default where cells with UMIs >= min_umi=3 are assigned the gRNA. Assign gRNAs to cells whose UMI counts satisfy the fitted threshold, then classify cells as singlets or multiplets, writing gRNA.umi.threshold.txt, consensus.sequence.matrix, cells.gRNA.txt, and cells.gRNA.single.txt.

## Related tools

- **GiRAFR** (Python package whose assign_gRNA.assign_gRNA_to_cell() module performs the thresholding and cell assignment) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **sklearn** (Fits the two-component Gaussian mixture model per gRNA to derive dynamic UMI thresholds (auto=True))
- **Cell Ranger** (Baseline tool whose fixed-threshold, mutation-blind assignment underestimates multiplets and labels mutant guides as intact; GiRAFR adopts its GMM strategy but is mutation-aware)

## Examples

```
from girafr import assign_gRNA; assign_gRNA.assign_gRNA_to_cell()
```

## Evaluation signals

- gRNA.umi.threshold.txt exists and contains exactly one threshold per gRNA in the library.
- cells.gRNA.txt and cells.gRNA.single.txt contain only cells whose gRNA UMI counts satisfy the fitted (or min_umi=3 fallback) threshold.
- With auto=True, thresholds vary per gRNA rather than being constant at min_umi=3; with auto disabled, assignments match the fixed min_umi=3 rule.
- Cells transduced with mutant gRNAs are retained in the assignment (pool=True), so the untransformed-cell fraction is not inflated relative to Cell Ranger.
- Singlet vs multiplet classification is consistent with cells.gRNA.single.txt containing only multiplicity-1 cells; GiRAFR shows up to 22% of Cell Ranger singlets are in fact multiplets.

## Limitations

- Inter-experiment differences in average gRNA UMI counts — whether from low gRNA expression or low detection rates — can detrimentally impact the accuracy of assigning gRNAs to cells and discriminating ambient from endogenously expressed guides.
- Prior studies use ad hoc customized pipelines and fixed thresholds, rendering head-to-head comparison of assignment results unfeasible; specific defined read depths must be attained for accurate gRNA assignment.
- Design choices such as multiplicity of infection, number of designed gRNAs, mutations in them, and number of cells per perturbation all influence assignment accuracy and require optimization or parameter finetuning.

## Evidence

- [methods] Fixed and dynamic UMI thresholds for assignment: "Integer number. By default = 3. Cells with UMIs >= min_umi will be assigned with gRNA."
- [readme] GiRAFR assigns gRNAs mutation-aware: "GiRAFR is a python package developed to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells in a sensitive, mutation-aware manner."
- [discussion] Ambient vs endogenous discrimination depends on UMI levels: "This may either be due to low gRNA expression or low detection rates, but can have a detrimental impact on the accuracy of assigning gRNAs to cells, discriminating ambient from endogenously expressed"
- [discussion] Cell Ranger underestimates multiplets: "up to 22% of all cells profiled in earlier studies that are labeled as singlets by Cell Ranger are in fact multiplets"
- [abstract] Consequence of ignoring mutant gRNAs in assignment: "failure to detect and annotate them leads to an inflated estimate of the number of untransformed cells, attenuated downregulation of target genes, as well as an underestimated multiplet frequency"
