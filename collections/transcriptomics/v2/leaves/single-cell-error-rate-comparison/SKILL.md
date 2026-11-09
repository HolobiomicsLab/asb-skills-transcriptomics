---
name: single-cell-error-rate-comparison
description: Use when multiple cell type identification methods must be compared on
  spatial transcriptomics data with known ground-truth single-cell labels, especially
  to test whether joint multi-slice analysis improves accuracy over slice-by-slice
  analysis.
when_to_use_negative:
- The ST data has no ground-truth single-cell labels, so error rate cannot be computed
  (use qualitative or deconvolution-only comparisons instead)
- The goal is deconvolution of cell-type proportions per spot rather than per-cell
  label accuracy against a gold standard
- Comparing methods on a single slice only, when the skill's purpose is to evaluate
  single-slice vs all-slices joint analysis
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2229
tools:
- name: SpatialScope
  role: Primary cell type identification method scored under single-slice and all-slices
    settings using its discrete RCTD-style probabilistic model with Potts prior nu
    = 10
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: RCTD (spacexr)
  role: Baseline run via StarDist+RCTD in full doublet mode; identified cell type
    mean gene expression used as per-cell profile
  repo: https://github.com/dmcable/spacexr
- name: Tangram
  role: Baseline alignment method; cell type labels recovered via its project_genes
    mapping per slice and across slices
- name: CytoSPACE
  role: Baseline alignment method; mapped single-cell IDs taken directly as the assigned
    labels for each spot
- name: StarDist
  role: Nucleus segmentation front-end producing per-cell inputs for the RCTD baseline
    (StarDist+RCTD)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:bd6ad86db86c2f12fdc2b7cac80c970bf0969e4b2a0aaa8332fbddb4a7d5f1de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - CytoSPACE
  - RCTD
  - RCTD (spacexr)
  - SpatialScope
  - StarDist
  - Tangram
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# single-cell-error-rate-comparison

## Summary

Benchmarks single-cell cell type identification accuracy across multiple spatial transcriptomics methods by scoring per-cell label assignments against ground-truth single-cell labels under single-slice and all-slices (joint) analysis settings, as done for SpatialScope vs Tangram, CytoSPACE, and StarDist+RCTD on simulated multi-slice Visium benchmark datasets.

## When to use

Use when multiple cell type identification methods must be compared on spatial transcriptomics data with known ground-truth single-cell labels, especially to test whether joint multi-slice analysis improves accuracy over slice-by-slice analysis.

## When NOT to use

- The ST data has no ground-truth single-cell labels, so error rate cannot be computed (use qualitative or deconvolution-only comparisons instead)
- The goal is deconvolution of cell-type proportions per spot rather than per-cell label accuracy against a gold standard
- Comparing methods on a single slice only, when the skill's purpose is to evaluate single-slice vs all-slices joint analysis

## Inputs

- Multi-slice simulated Visium benchmark datasets with ground-truth single-cell cell type labels (Datasets 5-6)
- Paired scRNA-seq reference data for each benchmark dataset
- Per-spot spatial coordinates for neighborhood/Potts modeling

## Outputs

- Per-method, per-dataset single-cell-level error rates under single-slice and all-slices settings
- cell_type_identification_error_rates.csv comparison table

## How to apply

Load multi-slice benchmark datasets with ground-truth single-cell labels (here Datasets 5-6, simulated Visium spots paired with scRNA-seq references) and run each competing method twice: once per slice individually and once jointly across all slices. Run SpatialScope Step 2 (cell type identification) with the discrete RCTD-style probabilistic model and Potts spatial smoothness prior at default nu = 10; run StarDist+RCTD with RCTD in full doublet mode using the mean gene expression of the identified cell type as the per-cell profile; recover Tangram labels via its project_genes mapping; take CytoSPACE mapped single-cell IDs as assigned spot labels. Score every method by single-cell-level error rate against ground-truth labels under both settings and assemble the comparison table. Correct application is confirmed when the error-rate table reproduces the expected pattern — e.g., SpatialScope reducing error rate by 22.9-50.0% on Dataset 5 and 4.6-48.3% on Dataset 6 relative to competitors, and being the only method whose error rate decreases under the all-slices setting.

## Related tools

- **SpatialScope** (Primary cell type identification method scored under single-slice and all-slices settings using its discrete RCTD-style probabilistic model with Potts prior nu = 10) — https://github.com/YangLabHKUST/SpatialScope
- **RCTD (spacexr)** (Baseline run via StarDist+RCTD in full doublet mode; identified cell type mean gene expression used as per-cell profile) — https://github.com/dmcable/spacexr
- **Tangram** (Baseline alignment method; cell type labels recovered via its project_genes mapping per slice and across slices)
- **CytoSPACE** (Baseline alignment method; mapped single-cell IDs taken directly as the assigned labels for each spot)
- **StarDist** (Nucleus segmentation front-end producing per-cell inputs for the RCTD baseline (StarDist+RCTD))

## Examples

```
git clone https://github.com/YangLabHKUST/SpatialScope.git && conda env create -f environment.yml  # then run SpatialScope Step 2 per-slice and all-slices on Datasets 5-6 and export cell_type_identification_error_rates.csv
```

## Evaluation signals

- Error rates are computed at the single-cell level against ground-truth labels for every method under both single-slice and all-slices settings
- Reproduced table shows SpatialScope reducing error rate by 22.9-50.0% on Dataset 5 and 4.6-48.3% on Dataset 6 versus competing methods
- Only SpatialScope's error rate decreases when moving from per-slice to all-slices joint analysis
- The comparison is saved as cell_type_identification_error_rates.csv with both settings represented for all methods

## Limitations

- Benchmark depends on simulated datasets with ground-truth labels; behavior on real data without ground truth cannot be scored this way
- The Potts model assumption that neighboring cells belong to the same cell type may not always be valid, which affects SpatialScope's scores
- Platform effects must be appropriately corrected or cell type identification results would not be satisfactory
- No software version numbers or computational environment details are reported, complicating exact reproduction

## Evidence

- [results] SpatialScope remained the most accurate method, reducing error rate by 22.9-50.0% on Dataset 5 and 4.6-48.3% on Dataset 6, and was the only method whose error rate decreased under the all-slices setting.: [evidence span withheld — anchor status: not_found]
- [methods] SpatialScope uses a discrete RCTD-style probabilistic model with a Potts spatial smoothness prior at default nu = 10 for cell type identification.: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [intro] Unlike alignment-based methods, SpatialScope does not merely reassign existing scRNA-seq cells to spots.: "unlike alignment-based methods such as Tangram18 and CytoSPACE21 that assign existing cells from scRNA-seq data to spatial spots"
- [readme] RCTD supports mixed pixels and platform effect normalization, making it a strong baseline for doublet-mode labeling.: [evidence span withheld — anchor status: external_doc]
- [discussion] Cell type identification results would be unsatisfactory if platform effects are not corrected.: "Cell type identification and gene expression decomposition results would not be satisfactory if the platform effects are not appropriately corrected."
