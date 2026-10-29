---
name: singlet-multiplet-classification
description: Use when a CROP-seq-style gRNA library must be assigned to cell barcodes
  from a filtered cell list, with singlet/multiplet distinction needed and per-guide
  UMI thresholds fitted automatically.
when_to_use_negative:
- Input is not a single-cell gRNA library with UMI counts per cell barcode (e.g. bulk
  perturbation sequencing without barcodes).
- You only need to profile gRNA library mutations (gRNA mutation profiling mode) rather
  than assign gRNAs to cells.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GiRAFR
  role: python package whose assign_gRNA.assign_gRNA_to_cell() step performs GMM-based
    UMI thresholding and singlet/multiplet assignment
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: pysam
  role: underlying pysam-based I/O layer GiRAFR is built on for reading gRNA BAM alignments
  repo: https://github.com/pysam-developers/pysam
- name: Cell Ranger
  role: baseline comparator whose fixed-threshold, mutation-blind singlet calls underestimate
    multiplets
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:117292b4ff0d9e7917afb35b54cc4e53c901371d488e41de633f7e72859bc928
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
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# singlet-multiplet-classification

## Summary

Assign intact and mutant gRNAs to single cells using per-guide UMI thresholding (fixed min_umi=3 or a two-component Gaussian mixture model) and classify cells as singlets or multiplets. This corrects the underestimated multiplet frequency and inflated untransformed-cell counts caused by mutation-blind tools like Cell Ranger.

## When to use

Use when a CROP-seq-style gRNA library must be assigned to cell barcodes from a filtered cell list, with singlet/multiplet distinction needed and per-guide UMI thresholds fitted automatically.

## When NOT to use

- Input is not a single-cell gRNA library with UMI counts per cell barcode (e.g. bulk perturbation sequencing without barcodes).
- You only need to profile gRNA library mutations (gRNA mutation profiling mode) rather than assign gRNAs to cells.

## Inputs

- per-UMI mutation call file consensus.sequence.gRNA.variant.txt
- filtered cell barcode list from expression library mapping
- optional per-guide gRNA references (Oligo_pool.csv with oligo_name and sequence columns)

## Outputs

- gRNA.umi.threshold.txt (one fitted threshold per gRNA)
- consensus.sequence.matrix
- cells.gRNA.txt (gRNA-to-cell assignments)
- cells.gRNA.single.txt (singlet assignments)

## How to apply

Load the per-UMI mutation calls from consensus.sequence.gRNA.variant.txt and the filtered cell barcode list from the expression library mapping. Pool intact and mutant UMI counts per gRNA (pool=True) so a single distribution per guide is modeled, then fit a two-component Gaussian mixture model with scikit-learn (auto=True) to separate endogenous in-cell UMIs from ambient UMIs; this dynamic threshold replaces the fixed min_umi=3 default. Assign gRNAs to cells whose UMI counts meet the fitted threshold (fall back to min_umi=3 when auto is disabled), then write gRNA.umi.threshold.txt, consensus.sequence.matrix, cells.gRNA.txt, and cells.gRNA.single.txt. The rationale is that mutation-blind assignment inflates singlet counts — up to 22% of cells labeled singlets by Cell Ranger are actually multiplets — so both mutant and intact gRNA UMIs must be included in the model.

## Related tools

- **GiRAFR** (python package whose assign_gRNA.assign_gRNA_to_cell() step performs GMM-based UMI thresholding and singlet/multiplet assignment) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **pysam** (underlying pysam-based I/O layer GiRAFR is built on for reading gRNA BAM alignments) — https://github.com/pysam-developers/pysam
- **Cell Ranger** (baseline comparator whose fixed-threshold, mutation-blind singlet calls underestimate multiplets)

## Examples

```
from girafr import assign_gRNA; assign_gRNA.assign_gRNA_to_cell()
```

## Evaluation signals

- cells.gRNA.txt and cells.gRNA.single.txt exist and are non-empty.
- cells.gRNA.txt contains only cells whose gRNA UMI counts satisfy the fitted threshold.
- gRNA.umi.threshold.txt contains exactly one threshold per gRNA.
- Multiplet frequency in cells.gRNA.txt is higher than the Cell Ranger baseline (mutation-blind tools underestimate multiplets, with up to 22% of Cell Ranger singlets actually multiplets).
- UMIs below min_umi=3 are excluded when the fixed threshold mode (auto disabled) is used.

## Limitations

- Inter-experiment differences in average gRNA UMIs — whether from low gRNA expression or low detection rates — can detrimentally impact discrimination of ambient vs endogenously expressed gRNAs and singlet vs multiplet calls.
- Design choices (multiplicity of infection, number of designed gRNAs, number of cells per perturbation) influence assignment accuracy and require optimization.
- Fixed min_umi=3 thresholds can be applied by mistake; ad hoc fixed thresholds across studies render head-to-head comparison unfeasible.

## Evidence

- [methods] Integer number. By default = 3. Cells with UMIs >= min_umi will be assigned with gRNA.: "Integer number. By default = 3. Cells with UMIs >= min_umi will be assigned with gRNA."
- [discussion] up to 22% of all cells profiled in earlier studies that are labeled as singlets by Cell Ranger are in fact multiplets: "up to 22% of all cells profiled in earlier studies that are labeled as singlets by Cell Ranger are in fact multiplets"
- [readme] failure to detect and annotate them leads to an inflated estimate of the number of untransformed cells as well as an underestimated multiplet frequency: "failure to detect and annotate them leads to an inflated estimate of the number of untransformed cells as well as an underestimated multiplet frequency"
