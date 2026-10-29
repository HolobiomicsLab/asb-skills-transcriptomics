---
name: simulated-visium-spot-analysis
description: Use when evaluating cell type identification accuracy on simulated multi-slice
  Visium spots where ground-truth per-cell labels exist and you need to test whether
  joint analysis of all slices outperforms per-slice analysis.
when_to_use_negative:
- The ST data are real image-based measurements (e.g., MERFISH) rather than simulated
  Visium spots with ground-truth labels — evaluation by single-cell error rate is
  impossible without a ground truth
- The goal is gene expression imputation or decomposition rather than discrete cell
  type label assignment
- Only a single slice is available, so the single-slice vs all-slices comparison is
  meaningless
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SpatialScope
  role: 'Primary method: Step 2 cell type identification with discrete RCTD-style
    probabilistic model and Potts prior (nu=10), run per-slice and jointly on all
    slices'
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: RCTD
  role: Baseline run via StarDist+RCTD in full doublet mode; per-cell profiles taken
    as the mean gene expression of the identified cell type
  repo: https://github.com/dmcable/spacexr
- name: Tangram
  role: Alignment-based baseline; cell type labels recovered via its project_genes
    mapping, per slice and across slices
- name: CytoSPACE
  role: Alignment-based baseline; mapped single-cell IDs serve as the assigned labels
    for each spot
- name: StarDist
  role: Nucleus segmentation (Step 1) feeding into the StarDist+RCTD baseline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:83c7757d8d04b67b890a33b0ae539dd296538e3f6403a41ad8039edebc908d83
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# simulated-visium-spot-analysis

## Summary

Benchmarks single-cell cell type identification on simulated multi-slice Visium datasets (Datasets 5-6) with ground-truth single-cell labels, comparing single-slice vs all-slices settings for SpatialScope against Tangram, CytoSPACE, and StarDist+RCTD.

## When to use

Use when evaluating cell type identification accuracy on simulated multi-slice Visium spots where ground-truth per-cell labels exist and you need to test whether joint analysis of all slices outperforms per-slice analysis.

## When NOT to use

- The ST data are real image-based measurements (e.g., MERFISH) rather than simulated Visium spots with ground-truth labels — evaluation by single-cell error rate is impossible without a ground truth
- The goal is gene expression imputation or decomposition rather than discrete cell type label assignment
- Only a single slice is available, so the single-slice vs all-slices comparison is meaningless

## Inputs

- Benchmarking Dataset 5 and Dataset 6: multi-slice simulated Visium spots with ground-truth single-cell labels
- Paired scRNA-seq reference data for each simulated dataset (from the SpatialScope demos directory)
- Paired histology images for nucleus segmentation (Step 1) where applicable

## Outputs

- Per-spot single-cell cell type labels for each method under single-slice and all-slices settings
- cell_type_identification_error_rates.csv: single-cell-level error-rate comparison table across SpatialScope, Tangram, CytoSPACE, and StarDist+RCTD

## How to apply

Load Benchmarking Datasets 5 and 6 (multi-slice simulated Visium spots with ground-truth single-cell labels) with their paired scRNA-seq references from the SpatialScope demos. Run SpatialScope Step 2 (cell type identification) per slice and jointly using the discrete RCTD-style probabilistic model with the Potts spatial smoothness prior at default nu = 10. Run StarDist+RCTD (RCTD full doublet mode, per-cell profiles = mean gene expression of the identified cell type), Tangram (labels via project_genes), and CytoSPACE (mapped single-cell IDs as spot labels) under both settings. Score all methods by single-cell-level error rate against ground truth; confirm SpatialScope reduces error rate by 22.9-50.0% on Dataset 5 and 4.6-48.3% on Dataset 6 and is the only method that improves in the all-slices setting.

## Related tools

- **SpatialScope** (Primary method: Step 2 cell type identification with discrete RCTD-style probabilistic model and Potts prior (nu=10), run per-slice and jointly on all slices) — https://github.com/YangLabHKUST/SpatialScope
- **RCTD** (Baseline run via StarDist+RCTD in full doublet mode; per-cell profiles taken as the mean gene expression of the identified cell type) — https://github.com/dmcable/spacexr
- **Tangram** (Alignment-based baseline; cell type labels recovered via its project_genes mapping, per slice and across slices)
- **CytoSPACE** (Alignment-based baseline; mapped single-cell IDs serve as the assigned labels for each spot)
- **StarDist** (Nucleus segmentation (Step 1) feeding into the StarDist+RCTD baseline)

## Examples

```
git clone https://github.com/YangLabHKUST/SpatialScope.git && cd SpatialScope && conda env create -f environment.yml  # then run Cell_Type_Identification.py on Datasets 5-6 per-slice and all-slices with nu=10
```

## Evaluation signals

- Error rate on Dataset 5 falls in the reported 22.9-50.0% reduction range for SpatialScope versus baselines
- Error rate on Dataset 6 falls in the reported 4.6-48.3% reduction range for SpatialScope versus baselines
- Only SpatialScope shows a decreased error rate when moving from single-slice to the all-slices setting; baselines do not improve
- Output CSV (cell_type_identification_error_rates.csv) contains error rates for all four methods under both settings with correct row/column schema
- SpatialScope run uses the discrete RCTD-style model with Potts hyperparameter nu = 10 by default

## Limitations

- The Potts model assumption that neighboring cells belong to the same cell type may not always be valid
- Training the deep generative models is time-consuming (hours or days)
- Results depend on appropriate correction of platform effects; unsatisfactory outcomes occur if platform effects are not corrected

## Evidence

- [results] SpatialScope remained the most accurate method for inferring single-cell cell type labels: [evidence span withheld — anchor status: not_found]
- [methods] Potts prior default: [evidence span withheld — anchor status: not_found]
- [methods] three-step pipeline: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [discussion] Potts assumption caveat: "the assumption that neighboring cells belong to the same cell type may not always be valid"
- [discussion] Platform effect caveat: "Cell type identification and gene expression decomposition results would not be satisfactory if the platform effects are not appropriately corrected."
