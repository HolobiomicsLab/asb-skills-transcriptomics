---
name: segmentation-quality-metric-evaluation
description: Use when you have segmented-cell masks (_connected.tif) and cell-gene
  matrices (expr_mat.csv) from a spatial transcriptomics run and need to quantify
  or compare segmentation quality across methods or ablation configurations.
when_to_use_negative:
- Input is raw Xenium/CosMx/MERSCOPE data that has not yet been segmented — run BIDCell
  preprocess/train/predict first to obtain masks and expr_mat.csv
- The goal is nuclei segmentation quality alone (e.g., validating Cellpose or Mesmer
  nuclear masks) rather than whole-cell body segmentation evaluated by transcript-assignment
  metrics
- Comparing segmentation-free transcript clustering methods (e.g., Baysor) when only
  their transcript assignments, not spatial masks with cell IDs, are available
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: BIDCell
  role: Produces the segmentation masks (_connected.tif) and cell-gene matrices (expr_mat.csv)
    that are evaluated; supports ablation by zeroing individual biologically-informed
    loss lambdas in the .yaml config
  repo: https://github.com/SydneyBioX/BIDCell
- name: 10x Genomics Xenium
  role: Source platform for the Xenium Output Bundle In Situ Replicate 1 breast cancer
    dataset used as the benchmarking input
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:88c82fd8f073d3df4db7f55c0292d117f4b8aece129adba732a94a931d84916b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - 10x Genomics Xenium
  - BIDCell
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# segmentation-quality-metric-evaluation

## Summary

Evaluate subcellular spatial transcriptomics cell segmentation outputs (e.g., BIDCell masks on Xenium-BreastCancer1) using CellSPA-style metrics — purity F1, captured transcripts, and Chromium correlation — including per-loss ablation comparisons.

## When to use

Use when you have segmented-cell masks (_connected.tif) and cell-gene matrices (expr_mat.csv) from a spatial transcriptomics run and need to quantify or compare segmentation quality across methods or ablation configurations.

## When NOT to use

- Input is raw Xenium/CosMx/MERSCOPE data that has not yet been segmented — run BIDCell preprocess/train/predict first to obtain masks and expr_mat.csv
- The goal is nuclei segmentation quality alone (e.g., validating Cellpose or Mesmer nuclear masks) rather than whole-cell body segmentation evaluated by transcript-assignment metrics
- Comparing segmentation-free transcript clustering methods (e.g., Baysor) when only their transcript assignments, not spatial masks with cell IDs, are available

## Inputs

- Segmented-cell mask .tif file ending in _connected.tif (pixel values correspond to cell IDs)
- Cell-gene matrix expr_mat.csv from BIDCell predict stage
- Xenium transcripts (transcripts.csv.gz) and DAPI image (morphology_mip.ome.tif) for reference metrics
- Per-configuration BIDCell .yaml parameter files (e.g., xenium_example_config.yaml with one loss lambda zeroed per ablation)

## Outputs

- ablation_metrics.csv — tabular file of purity F1, captured transcripts, and Chromium correlation per configuration
- Per-configuration metric values ordered to match trends in Supplementary Figs. 4-5

## How to apply

Collect each run's outputs — the segmented-cell mask ending in `_connected.tif` (pixel values are cell IDs) and the cell-gene matrix `expr_mat.csv` produced by `model.predict()` and `make_cell_gene_mat()`. Evaluate each configuration with CellSPA-style metric categories spanning five complementary criteria, computing purity F1, the amount of captured transcripts, and Chromium correlation, as used for Supplementary Figs. 4-5. For loss-function ablations, zero one biologically-informed loss weight lambda in the .yaml (and disable shape-specific expansions), retrain ~4,000 steps, and re-evaluate so the relative ordering of configurations can be checked against reported degradation trends. Assemble all per-configuration metric values into a single tabular file `ablation_metrics.csv`. Correct application is judged by reproducing the expected pattern: removing individual biologically-informed losses degrades performance, marked by increased purity F1 relative to captured transcripts when losses are combined.

## Related tools

- **BIDCell** (Produces the segmentation masks (_connected.tif) and cell-gene matrices (expr_mat.csv) that are evaluated; supports ablation by zeroing individual biologically-informed loss lambdas in the .yaml config) — https://github.com/SydneyBioX/BIDCell
- **10x Genomics Xenium** (Source platform for the Xenium Output Bundle In Situ Replicate 1 breast cancer dataset used as the benchmarking input)

## Examples

```
from bidcell import BIDCellModel; model = BIDCellModel('xenium_example_config.yaml'); model.preprocess(); model.train(); model.predict()
```

## Evaluation signals

- Ablation ordering reproduces Supplementary Figs. 4-5 trends: performance degrades when key biologically-informed loss components are removed
- Combining losses yields a marked change in purity F1 relative to the amount of captured transcripts versus single-loss removals
- Each output mask file is a valid .tif ending in _connected.tif with pixel values corresponding to cell IDs
- All per-configuration metrics are present in ablation_metrics.csv with no missing configurations
- Chromium correlation is computable for every configuration's expr_mat.csv against the reference expression profiles

## Limitations

- Achieving more homogenous cell bodies can result from arbitrary over-segmentation of nuclei, so a single metric can be misleading and multiple CellSPA metric categories are needed
- In MERSCOPE data with cell membrane markers, ~25% of cells lack nuclei in their segmentation, complicating nucleus-based comparisons
- There is a lack of ground truth annotations for SST segmentation; manual annotation of individual pixels is unattainable, so metrics are indirect proxies for segmentation correctness

## Evidence

- [intro] Evaluation metrics and CellSPA categories: "we created a comprehensive evaluation framework for cell segmentation, CellSPA, that assesses five complementary categories of criteria for identifying the optimal segmentation strategies"
- [readme] Expected output files: ".tif file of segmented cells, where the value corresponds to cell IDs. File name ends in `_connected.tif`"
- [other] Ablation finding: [evidence span withheld — anchor status: not_found]
- [other] Metric-collection workflow: [evidence span withheld — anchor status: not_found]
