---
name: multi-slice-spatial-alignment-evaluation
description: Use when multiple adjacent or replicated ST slices (e.g., simulated Visium
  Datasets 5-6) with paired scRNA-seq references are available and you need to decide
  whether analyzing slices jointly improves cell type labeling accuracy versus slice-by-slice
  analysis.
when_to_use_negative:
- Only a single ST slice is available, so the joint-vs-individual comparison is undefined
- No ground-truth single-cell labels exist, making single-cell-level error-rate scoring
  impossible (use deconvolution proportion metrics instead)
- The goal is gene expression imputation or decomposition rather than discrete cell
  type label assignment
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: SpatialScope
  role: Primary cell type identification method run per-slice and jointly on all slices
    with RCTD-style discrete model and Potts prior nu=10
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: Tangram
  role: Alignment-based baseline; cell type labels recovered via project_genes mapping
    per-slice and across slices
- name: CytoSPACE
  role: Alignment-based baseline; mapped single-cell IDs taken as assigned labels
    for each spot
- name: RCTD
  role: Baseline run in full doublet mode via StarDist+RCTD, using mean gene expression
    of identified cell type as per-cell profile
  repo: https://github.com/dmcable/spacexr
- name: StarDist
  role: Nucleus segmentation step feeding RCTD in the StarDist+RCTD baseline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:5f046e228b8e7e96a85331d17234266d58497f3e3d2ea5539117c7dd0dad315d
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
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# multi-slice-spatial-alignment-evaluation

## Summary

Benchmarks single-cell cell type identification on multiple spatial transcriptomics slices by running methods (SpatialScope, Tangram, CytoSPACE, StarDist+RCTD) per-slice and jointly on all slices, then scoring single-cell-level error rates against ground-truth labels to test whether joint multi-slice inference improves accuracy.

## When to use

Use when multiple adjacent or replicated ST slices (e.g., simulated Visium Datasets 5-6) with paired scRNA-seq references are available and you need to decide whether analyzing slices jointly improves cell type labeling accuracy versus slice-by-slice analysis.

## When NOT to use

- Only a single ST slice is available, so the joint-vs-individual comparison is undefined
- No ground-truth single-cell labels exist, making single-cell-level error-rate scoring impossible (use deconvolution proportion metrics instead)
- The goal is gene expression imputation or decomposition rather than discrete cell type label assignment

## Inputs

- Multi-slice simulated Visium benchmark datasets (Datasets 5-6) with spot coordinates, counts, and ground-truth single-cell labels
- Paired scRNA-seq reference matrices with cell type annotations
- Paired histological images (for StarDist nucleus segmentation feeding RCTD)

## Outputs

- Per-method, per-slice, and per-setting single-cell-level error rates
- cell_type_identification_error_rates.csv comparison table (single-slice vs all-slices settings across SpatialScope, Tangram, CytoSPACE, StarDist+RCTD)

## How to apply

Load the multi-slice benchmark datasets (Datasets 5-6, simulated Visium spots with ground-truth single-cell labels) plus their paired scRNA-seq references. Run SpatialScope Step 2 (cell type identification) on each slice individually and then jointly on all slices, using the discrete RCTD-style probabilistic model with the Potts spatial smoothness prior at default nu = 10. Run StarDist+RCTD (RCTD in full doublet mode, using mean gene expression of the identified cell type as the per-cell profile), Tangram (recovering labels via its project_genes mapping), and CytoSPACE (taking its mapped single-cell IDs as spot labels) under both settings. Score all methods by single-cell-level error rate against ground-truth labels per slice and per setting, and assemble the comparison table. The expected outcome is that SpatialScope remains most accurate (error rate reduced 22.9-50.0% on Dataset 5 and 4.6-48.3% on Dataset 6) and is the only method whose error rate decreases under the all-slices setting.

## Related tools

- **SpatialScope** (Primary cell type identification method run per-slice and jointly on all slices with RCTD-style discrete model and Potts prior nu=10) — https://github.com/YangLabHKUST/SpatialScope
- **Tangram** (Alignment-based baseline; cell type labels recovered via project_genes mapping per-slice and across slices)
- **CytoSPACE** (Alignment-based baseline; mapped single-cell IDs taken as assigned labels for each spot)
- **RCTD** (Baseline run in full doublet mode via StarDist+RCTD, using mean gene expression of identified cell type as per-cell profile) — https://github.com/dmcable/spacexr
- **StarDist** (Nucleus segmentation step feeding RCTD in the StarDist+RCTD baseline)

## Examples

```
git clone https://github.com/YangLabHKUST/SpatialScope.git; cd SpatialScope; conda env create -f environment.yml  # then run Step 2 cell type identification per-slice and all-slices with nu=10
```

## Evaluation signals

- SpatialScope error rates on Dataset 5 should fall 22.9-50.0% below baselines and 4.6-48.3% on Dataset 6
- Only SpatialScope's error rate should decrease when switching from single-slice to all-slices setting; baselines should not improve
- Error rates are computed at single-cell level against ground-truth labels, not spot-level proportions
- The output table cell_type_identification_error_rates.csv must contain rows for all four methods under both settings for both datasets

## Limitations

- The Potts model assumption that neighboring cells belong to the same cell type may not always be valid
- Training the deep generative models is time-consuming, measured in hours or days
- Cell type identification results would not be satisfactory if platform effects are not appropriately corrected

## Evidence

- [results] Multi-slice benchmark finding: [evidence span withheld — anchor status: not_found]
- [methods] Potts prior default: [evidence span withheld — anchor status: not_found]
- [discussion] Potts assumption caveat: "the assumption that neighboring cells belong to the same cell type may not always be valid"
- [readme] RCTD platform effects: [evidence span withheld — anchor status: external_doc]
- [methods] SpatialScope three steps: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
