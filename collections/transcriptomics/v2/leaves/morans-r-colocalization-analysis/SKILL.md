---
name: morans-r-colocalization-analysis
description: Use when you have spatially reconstructed (or truth) single-cell coordinates
  with cell-type annotations and need to detect which cell-type pairs are significantly
  co-localized, e.g. after CellContrast spatial inference.
when_to_use_negative:
- Input SC data has no reconstructed or reference spatial coordinates — Moran's R
  requires locations, so use a spatial reconstruction skill first
- Tissue has a high level of repeating elements or a large gap between SC and ST reference,
  where spatial reconstruction itself is unreliable and co-localization calls would
  inherit the bias
- You only need cell-cell communication inference rather than cell-type spatial co-localization
  patterns
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: CellContrast
  role: Provides the spatially reconstructed SC locations on which Moran's R co-localization
    is computed
  repo: https://github.com/HKU-BAL/CellContrast
- name: SpatialDM
  role: Runs Moran's R spatial autocorrelation test on binary cell-type occupancy
    to call significant co-localization pairs
- name: CeLEry
  role: Alternative spatial reconstruction whose coordinates are tested with the same
    Moran's R workflow for cross-method comparison
- name: SageNet
  role: MDS-projects predicted cell-cell distances into pseudo space whose coordinates
    are tested with Moran's R (l=1e-4)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2024.101022
    title: 'CellContrast: Reconstructing spatial relationships in single-cell RNA
      sequencing data via deep contrastive learning'
  source_content_hash: sha256:a126bfaf0a6d418db4bcbff1e4aad941dc5d6e1198896dc9e0ae5976b9812674
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HKU-BAL/CellContrast
  tools:
  - CeLEry
  - CellContrast
  - SageNet
  - SpatialDM
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# morans-r-colocalization-analysis

## Summary

Detects cell-type co-localization pairs from reconstructed spatial locations using SpatialDM Moran's R spatial autocorrelation, encoding binary cell-type occupancy per location and calling significant pairs at p<=0.05 for downstream Fisher's exact comparison against reference patterns.

## When to use

Use when you have spatially reconstructed (or truth) single-cell coordinates with cell-type annotations and need to detect which cell-type pairs are significantly co-localized, e.g. after CellContrast spatial inference.

## When NOT to use

- Input SC data has no reconstructed or reference spatial coordinates — Moran's R requires locations, so use a spatial reconstruction skill first
- Tissue has a high level of repeating elements or a large gap between SC and ST reference, where spatial reconstruction itself is unreliable and co-localization calls would inherit the bias
- You only need cell-cell communication inference rather than cell-type spatial co-localization patterns

## Inputs

- Spatially reconstructed SC coordinates as AnnData (.h5ad), e.g. spatial_reconstructed_sc.h5ad with reconstructed x/y locations in .uns
- Cell-type annotations per cell for encoding binary occupancy
- Reference SeqFISH ST datasets with truth cell locations for building reference co-localization patterns

## Outputs

- Set of significant cell-type co-localization pairs per method at p<=0.05 (e.g. 42 pairs for CellContrast/CeLEry, 38 for SageNet)
- Fisher's exact test p-values comparing detected vs reference co-localizations (e.g. 2.793e-9, 1.121e-6, 7.409e-6)
- Co-localization results table, e.g. colocalization_results.csv

## How to apply

First obtain reconstructed cell coordinates, e.g. the spatial_reconstructed_sc.h5ad produced by CellContrast inference on a query SC sample (such as mouse SC atlas S27, E8.5) mapped onto a SeqFISH embryo ST reference. Encode a binary cell-type occupancy variable (0 or 1) per location for each cell type. Run SpatialDM Moran's R with n_neighbors=20 and a radial basis kernel with length scale l=0.1 for CellContrast and CeLEry coordinates, and l=1e-4 for SageNet MDS pseudo-space, calling significant co-localizations at p<=0.05. Filter detected cell-type pairs to those with reference support >1 from the SeqFISH reference datasets, then compare the counts (e.g. 42 pairs for CellContrast, 38 for SageNet) and compute Fisher's exact tests between detected and reference co-localizations. Save counts and p-values (e.g. colocalization_results.csv) for cross-method comparison.

## Related tools

- **CellContrast** (Provides the spatially reconstructed SC locations on which Moran's R co-localization is computed) — https://github.com/HKU-BAL/CellContrast
- **SpatialDM** (Runs Moran's R spatial autocorrelation test on binary cell-type occupancy to call significant co-localization pairs)
- **CeLEry** (Alternative spatial reconstruction whose coordinates are tested with the same Moran's R workflow for cross-method comparison)
- **SageNet** (MDS-projects predicted cell-cell distances into pseudo space whose coordinates are tested with Moran's R (l=1e-4))

## Examples

```
python cellContrast.py inference --ref_data_path train_ST.h5ad --query_data_path query_sc.h5ad --model_folder cellContrast_models/ --save_path spatial_reconstructed_sc.h5ad
```

## Evaluation signals

- Reproduced co-localization counts equal 42 (CellContrast), 42 (CeLEry), and 38 (SageNet) pairs
- Fisher's exact p-values match 2.793e-9 (CellContrast), 1.121e-6 (CeLEry), and 7.409e-6 (SageNet) within reasonable numerical tolerance
- 27 co-localization pairs are consistently detected across all three methods
- All retained pairs have reference support >1 from the six SeqFISH reference datasets
- Significant calls satisfy p<=0.05 under SpatialDM Moran's R with n_neighbors=20

## Limitations

- SC-ST mapping introduces a 'cell clumping' issue where multiple neighbors are projected into the same reference location, causing overlap and loss of predicted SC-SC distances
- MDS-based mapping (used for SageNet pseudo-space) loses the pre-knowledge of the reference's cell distribution
- A certain level of mismatch may exist between the query SC data and the ST reference due to limited ST resources
- Tissues with a high level of repeating elements could pose a challenge for spatial reconstruction of SC cells

## Evidence

- [discussion] we reconstructed spatial locations for cells from a mouse embryo on E8.5 and detected cell-type co-localization patterns with Moran’s R: "we reconstructed spatial locations for cells from a mouse embryo on E8.5 and detected cell-type co-localization patterns with Moran’s R"
- [full_text] Encode binary occupancy and run SpatialDM: [evidence span withheld — anchor status: not_found]
- [discussion] Reference patterns from six ST datasets: "To validate our findings, we built reference patterns from six ST datasets with truth cell locations."
- [readme] README spatial inference output: "## Output file: spatially reconstructed h5ad file of annData"
