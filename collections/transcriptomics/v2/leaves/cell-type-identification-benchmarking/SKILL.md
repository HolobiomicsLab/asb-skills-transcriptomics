---
name: cell-type-identification-benchmarking
description: Use when evaluating whether joint (all-slices) inference improves cell
  type labeling accuracy of spatial transcriptomics methods, or when comparing ST
  cell type identification methods on simulated Visium data with known ground-truth
  single-cell labels.
when_to_use_negative:
- Input ST data lacks ground-truth single-cell labels, so single-cell-level error
  rate cannot be scored (use spot-level deconvolution metrics instead)
- The goal is gene expression imputation or decomposition rather than discrete cell
  type label assignment
- Data are image-based ST (e.g., MERFISH/seqFISH) rather than seq-based simulated
  Visium spots, since the benchmark protocol is built around Visium spot-level simulation
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SpatialScope
  role: Primary method under evaluation; Step 2 cell type identification run per slice
    and jointly with Potts prior nu = 10
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: Tangram
  role: Baseline comparator; cell type labels recovered via its project_genes mapping,
    run per slice and across slices
- name: CytoSPACE
  role: Baseline comparator; mapped single-cell IDs taken as assigned cell type labels
    per spot
- name: RCTD
  role: Baseline comparator run as StarDist+RCTD in full doublet mode; mean gene expression
    of the identified cell type used as the per-cell profile
  repo: https://github.com/dmcable/spacexr
- name: StarDist
  role: Nucleus segmentation paired with RCTD for the StarDist+RCTD baseline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:805f6928898b4aa3f2ffe4f1ff946709f605c4f10ef86a73360aeb8959909e68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - CytoSPACE
  - RCTD
  - SpatialScope
  - StarDist
  - Tangram
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cell-type-identification-benchmarking

## Summary

Benchmark single-cell cell type identification accuracy of SpatialScope against Tangram, CytoSPACE, and StarDist+RCTD on multi-slice simulated Visium datasets (Datasets 5-6), comparing single-slice vs all-slices inference settings using single-cell-level error rate against ground-truth labels.

## When to use

Use when evaluating whether joint (all-slices) inference improves cell type labeling accuracy of spatial transcriptomics methods, or when comparing ST cell type identification methods on simulated Visium data with known ground-truth single-cell labels.

## When NOT to use

- Input ST data lacks ground-truth single-cell labels, so single-cell-level error rate cannot be scored (use spot-level deconvolution metrics instead)
- The goal is gene expression imputation or decomposition rather than discrete cell type label assignment
- Data are image-based ST (e.g., MERFISH/seqFISH) rather than seq-based simulated Visium spots, since the benchmark protocol is built around Visium spot-level simulation

## Inputs

- Benchmarking Datasets 5-6: multi-slice simulated Visium spots with ground-truth single-cell cell type labels (from SpatialScope demos directory)
- Paired scRNA-seq reference data for each benchmark dataset (labeled cell type profiles)
- Per-slice and all-slices configurations of each ST dataset

## Outputs

- cell_type_identification_error_rates.csv: error-rate comparison table for SpatialScope, Tangram, CytoSPACE, and StarDist+RCTD under single-slice vs all-slices settings
- Per-cell predicted cell type labels for each method on each slice

## How to apply

Load Benchmarking Datasets 5-6 (multi-slice simulated Visium spots with ground-truth single-cell labels) and their paired scRNA-seq references from the SpatialScope demos directory. Run SpatialScope Step 2 (cell type identification) per slice and jointly, using the discrete RCTD-style probabilistic model with the Potts spatial smoothness prior at the default hyperparameter nu = 10. Run StarDist+RCTD (RCTD in full doublet mode, using mean gene expression of the identified cell type as the per-cell profile), Tangram (recovering labels via its project_genes mapping), and CytoSPACE (taking its mapped single-cell IDs as per-spot labels) under both settings. Score all methods by single-cell-level error rate against ground truth, then assemble the comparison table confirming SpatialScope reduces error rates by 22.9-50.0% on Dataset 5 and 4.6-48.3% on Dataset 6, and is the only method whose error rate decreases under the all-slices setting; save as cell_type_identification_error_rates.csv.

## Related tools

- **SpatialScope** (Primary method under evaluation; Step 2 cell type identification run per slice and jointly with Potts prior nu = 10) — https://github.com/YangLabHKUST/SpatialScope
- **Tangram** (Baseline comparator; cell type labels recovered via its project_genes mapping, run per slice and across slices)
- **CytoSPACE** (Baseline comparator; mapped single-cell IDs taken as assigned cell type labels per spot)
- **RCTD** (Baseline comparator run as StarDist+RCTD in full doublet mode; mean gene expression of the identified cell type used as the per-cell profile) — https://github.com/dmcable/spacexr
- **StarDist** (Nucleus segmentation paired with RCTD for the StarDist+RCTD baseline)

## Examples

```
git clone https://github.com/YangLabHKUST/SpatialScope.git && cd SpatialScope && conda env create -f environment.yml
```

## Evaluation signals

- SpatialScope error rate reductions of 22.9-50.0% on Dataset 5 and 4.6-48.3% on Dataset 6 are reproduced in the comparison table
- SpatialScope is the only method whose error rate decreases under the all-slices setting relative to single-slice
- All four methods are scored by single-cell-level error rate against the ground-truth labels under both settings
- Output table cell_type_identification_error_rates.csv contains rows for SpatialScope, Tangram, CytoSPACE, and StarDist+RCTD across both datasets and both settings
- SpatialScope cell type identification uses the discrete RCTD-style probabilistic model with Potts prior nu = 10 (default hyperparameter)

## Limitations

- The Potts model assumption that neighboring cells belong to the same cell type may not always be valid
- Platform effects must be appropriately corrected or cell type identification results will not be satisfactory
- Training the deep generative models is time-consuming (hours or days)

## Evidence

- [results] Multi-slice benchmark findings: [evidence span withheld — anchor status: not_found]
- [results] All-slices improvement: [evidence span withheld — anchor status: not_found]
- [methods] Potts prior parameter: [evidence span withheld — anchor status: not_found]
- [methods] Cell type identification step: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [intro] Comparison methods context: "unlike alignment-based methods such as Tangram18 and CytoSPACE21 that assign existing cells from scRNA-seq data to spatial spots"
