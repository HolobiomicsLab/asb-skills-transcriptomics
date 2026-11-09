---
name: single-cell-expression-preprocessing
description: Use when single-cell count matrices must be normalized, transformed,
  or batch-corrected before building 4D communication tensors, and robustness of inferred
  CCC patterns to preprocessing choices needs to be quantified.
when_to_use_negative:
- When only a single preprocessing strategy is of interest and no comparison of decomposition
  robustness across representations is needed
- When the dataset contains cell types not shared across all samples and rare cell
  types must be retained — the tensor construction excludes cell types missing in
  any context
- When the goal is per-context interaction inference rather than cross-context tensor
  decomposition
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: cell2cell
  role: Provides cell2cell.preprocessing functions for data preprocessing, normalization,
    and integration (ComBat, Scanorama), and cell2cell.stats utilities for computing
    1 - CorrIndex factor similarities
  repo: https://github.com/earmingol/cell2cell
- name: Tensor-cell2cell
  role: Decomposes each preprocessed 4D-communication tensor via tensor component
    analysis so decompositions from different representations can be compared
  repo: https://github.com/earmingol/cell2cell
- name: Ligand-Receptor Pairs curated list
  role: Source of the curated human LR pair list (Human-2020-Jin-LR-pairs.csv) used
    to score communication per preprocessing strategy
  repo: https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:0c270143a9f1588b526a5e41b92c5bde42eb0ef9e059eecfe705e6c57fd02937
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - Ligand-Receptor Pairs curated list
  - Tensor-cell2cell
  - cell2cell
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-expression-preprocessing

## Summary

Generate multiple expression representations of single-cell count data (raw counts, log(CPM+1), fraction of non-zero cells, ComBat and Scanorama batch-corrected matrices) with cell2cell.preprocessing, then assess how consistent the downstream Tensor-cell2cell communication decompositions remain across them.

## When to use

Use when single-cell count matrices must be normalized, transformed, or batch-corrected before building 4D communication tensors, and robustness of inferred CCC patterns to preprocessing choices needs to be quantified.

## When NOT to use

- When only a single preprocessing strategy is of interest and no comparison of decomposition robustness across representations is needed
- When the dataset contains cell types not shared across all samples and rare cell types must be retained — the tensor construction excludes cell types missing in any context
- When the goal is per-context interaction inference rather than cross-context tensor decomposition

## Inputs

- Single-cell count matrix: BALF COVID-19 dataset, 63k immune and epithelial cells across 3 control, 3 moderate, 6 severe patients
- Curated human ligand-receptor pair list (Jin et al. 2021; Human-2020-Jin-LR-pairs.csv)
- Sample/context metadata (control/moderate/severe labels) for batch-correction grouping

## Outputs

- Five expression representations per context: raw counts, log(CPM+1), non-zero fraction, ComBat-corrected, Scanorama-corrected
- Five 4D-communication tensors (contexts x LR pairs x senders x receivers)
- Full pairwise (1 - CorrIndex) similarity matrix written to corrindex_similarities.csv

## How to apply

Load the BALF COVID-19 single-cell counts (63k immune and epithelial cells from 3 control, 3 moderate, and 6 severe patients) and the curated human ligand-receptor list (Jin et al. 2021) using cell2cell.preprocessing. Produce five expression representations per context: raw counts, log(CPM+1) normalization, fraction of non-zero cells, ComBat batch-corrected, and Scanorama batch-corrected/integrated matrices. For each representation, build a 4D-communication tensor (contexts x LR pairs x sender cell types x receiver cell types), keeping only cell types shared across all samples, and decompose it with Tensor-cell2cell tensor component analysis (tensorly backend; switch init='svd' to init='random' if memory errors arise). Compare all pairwise decompositions as 1 - CorrIndex using cell2cell.stats utilities, then summarize the mean similarity across strategies and between raw counts and all other approaches.

## Related tools

- **cell2cell** (Provides cell2cell.preprocessing functions for data preprocessing, normalization, and integration (ComBat, Scanorama), and cell2cell.stats utilities for computing 1 - CorrIndex factor similarities) — https://github.com/earmingol/cell2cell
- **Tensor-cell2cell** (Decomposes each preprocessed 4D-communication tensor via tensor component analysis so decompositions from different representations can be compared) — https://github.com/earmingol/cell2cell
- **Ligand-Receptor Pairs curated list** (Source of the curated human LR pair list (Human-2020-Jin-LR-pairs.csv) used to score communication per preprocessing strategy) — https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs

## Examples

```
conda create -n cell2cell -y python=3.7 jupyter; conda activate cell2cell; pip install cell2cell
```

## Evaluation signals

- Mean pairwise (1 - CorrIndex) similarity across the five preprocessing strategies is approximately 0.86
- Mean similarity between the raw-counts decomposition and all other strategies is approximately 0.77
- Full pairwise similarity matrix is written to corrindex_similarities.csv with entries for all 5x5 strategy pairs
- Each tensor decomposition succeeds on the tensorly backend (use init='random' rather than init='svd' if memory errors occur)
- Each of the five representations yields a 4D tensor with cell types shared across all 12 patients (3 control, 3 moderate, 6 severe)

## Limitations

- Raw counts are the outlier: their decomposition similarity to all other approaches drops to ~0.77, so raw-count tensors should not be expected to match normalized results
- Only cell types shared across all samples can be included in the tensors, potentially missing rare cell types
- The implemented tensor decomposition algorithm is not optimized for missing values in the communication tensor
- Reproducibility depends on the exact code and software versions deposited in the Code Ocean capsule

## Evidence

- [other] Tensor-cell2cell detects consistent cell-cell communication signatures across log(CPM+1), non-zero fraction, ComBat, and Scanorama preprocessing: [evidence span withheld — anchor status: not_found]
- [methods] cell2cell.preprocessing module description: "cell2cell.preprocessing : Includes functions for manipulating data and variables (e.g. data preprocessing, integration, permutation, among others)."
- [discussion] Shared-cell-type filter for tensors: "we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases"
- [results] LR pair list source: "https://raw.githubusercontent.com/LewisLabUCSD/Ligand-Receptor-Pairs/master/Human/Human-2020-Jin-LR-pairs.csv"
- [discussion] Missing-value limitation: "the implemented tensor decomposition algorithm can be further optimized for missing values. Since the implemented algorithm is not optimized for this purpose"
