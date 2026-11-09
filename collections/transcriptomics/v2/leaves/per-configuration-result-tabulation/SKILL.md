---
name: per-configuration-result-tabulation
description: Use when multiple segmentation configurations (e.g., loss-weight ablations)
  have been run and per-run metric values must be collected, tabulated, and compared
  against reference trends.
when_to_use_negative:
- Only a single segmentation configuration was run, so there is no cross-configuration
  ordering to tabulate or compare
- The task is benchmarking BIDCell against external methods (e.g., Cellpose, Baysor)
  rather than comparing internal loss-ablation configurations
tools:
- name: BIDCell
  role: Produces the per-configuration segmentation outputs (_connected.tif masks
    and expr_mat.csv) and the CellSPA evaluation metrics that are tabulated
  repo: https://github.com/SydneyBioX/BIDCell
- name: 10x Genomics Xenium
  role: Source platform of the Xenium-BreastCancer1 data on which the ablation runs
    are evaluated
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:38dee71319dc54befcdd3c24f80398dc0691b9a87d1f71e4785184ab90c84d57
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - 10x Genomics Xenium
  - BIDCell
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# per-configuration-result-tabulation

## Summary

Assembles segmentation metric values (purity F1, captured transcripts, Chromium correlation) from each BIDCell ablation configuration into a single tabular file (ablation_metrics.csv) and verifies that the relative ordering of configurations reproduces the degradation trends reported in Supplementary Figs. 4-5.

## When to use

Use when multiple segmentation configurations (e.g., loss-weight ablations) have been run and per-run metric values must be collected, tabulated, and compared against reference trends.

## When NOT to use

- Only a single segmentation configuration was run, so there is no cross-configuration ordering to tabulate or compare
- The task is benchmarking BIDCell against external methods (e.g., Cellpose, Baysor) rather than comparing internal loss-ablation configurations

## Inputs

- Segmented-cell mask .tif files (ending in _connected.tif) from each ablation run
- Cell-gene matrix expr_mat.csv from each ablation run
- Reference configuration ~4,000 training steps per run from xenium_example_config.yaml

## Outputs

- ablation_metrics.csv containing per-configuration purity F1, captured transcripts, and Chromium correlation values

## How to apply

After each ablation run (one biologically-informed loss lambda zeroed in the xenium_example_config.yaml), collect the outputs: the segmented-cell mask ending in _connected.tif and the cell-gene matrix expr_mat.csv. Evaluate each configuration with the CellSPA-style metrics — purity F1, amount of captured transcripts, and Chromium correlation — used for Supplementary Figs. 4-5. Write all per-configuration metric values into a single tabular file, ablation_metrics.csv, with one row (or column block) per configuration. Then check that the relative ordering of configurations reproduces the reported trend: removing individual biologically-informed losses degrades performance, shown by a marked increase in purity F1 relative to the amount of captured transcripts when the losses were combined.

## Related tools

- **BIDCell** (Produces the per-configuration segmentation outputs (_connected.tif masks and expr_mat.csv) and the CellSPA evaluation metrics that are tabulated) — https://github.com/SydneyBioX/BIDCell
- **10x Genomics Xenium** (Source platform of the Xenium-BreastCancer1 data on which the ablation runs are evaluated)

## Examples

```
model = BIDCellModel("xenium_example_config.yaml"); model.preprocess(); model.train(); model.predict()  # per config, then compile metrics into ablation_metrics.csv
```

## Evaluation signals

- ablation_metrics.csv contains one entry per ablation configuration with all three metrics (purity F1, captured transcripts, Chromium correlation) populated
- The full-loss (combined losses) configuration shows lower purity F1 relative to the amount of captured transcripts compared with loss-removed configurations
- The relative ordering of configurations matches the degradation patterns shown in Supplementary Figs. 4-5
- Each configuration's _connected.tif and expr_mat.csv outputs exist and are non-empty before metric computation

## Limitations

- Achieving more homogeneous cell bodies (better purity F1) can also result from arbitrary over-segmentation of nuclei, motivating use of multiple complementary metrics rather than purity F1 alone
- Comparisons are only meaningful when all runs share identical preprocessing inputs (segment_nuclei, generate_expression_maps, generate_patches, make_cell_gene_mat, preannotate run once) and the same training-step budget (~4,000 steps)

## Evidence

- [other] Collect each run's outputs, including the segmented-cell mask ending in _connected.tif and the cell-gene matrix expr_mat.csv.: [evidence span withheld — anchor status: not_found]
- [other] Evaluate each configuration's segmentation with purity F1, captured transcripts, and Chromium correlation, following the CellSPA-style metric categories: [evidence span withheld — anchor status: not_found]
- [intro] we created a comprehensive evaluation framework for cell segmentation, CellSPA, that assesses five complementary categories of criteria: "we created a comprehensive evaluation framework for cell segmentation, CellSPA, that assesses five complementary categories of criteria for identifying the optimal segmentation strategies"
- [readme] .tif file of segmented cells, where the value corresponds to cell IDs. File name ends in _connected.tif: ".tif file of segmented cells, where the value corresponds to cell IDs. File name ends in `_connected.tif`"
- [discussion] While achieving more homogenous cell bodies is desired, it can also result from the arbitrary over-segmentation of nuclei.: "While achieving more homogenous cell bodies is desired, it can also result from the arbitrary over-segmentation of nuclei."
