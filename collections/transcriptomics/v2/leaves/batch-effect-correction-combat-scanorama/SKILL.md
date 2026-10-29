---
name: batch-effect-correction-combat-scanorama
description: Use when building context-wise 4D-communication tensors from multi-donor
  single-cell data (e.g. 12 BALF COVID-19 patients) where batch effects across samples
  could confound LR communication scores, and you need batch-corrected representations
  before tensor decomposition.
when_to_use_negative:
- Single-context data with no multi-sample/batch structure — correction is unnecessary
  before a per-context communication analysis
- When the goal is to preserve absolute count-level signal rather than cross-sample
  patterns, since raw-count decompositions are demonstrably less consistent (0.77
  vs 0.86)
- Cell types not shared across all samples, which are excluded from the tensor in
  this workflow
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: cell2cell
  role: Provides cell2cell.preprocessing for generating raw, log(CPM+1), non-zero
    fraction, ComBat, and Scanorama representations, and cell2cell.stats for CorrIndex-based
    factor similarity comparisons
  repo: https://github.com/earmingol/cell2cell
- name: Tensor-cell2cell
  role: Decomposes each batch-corrected 4D-communication tensor into interpretable
    factors whose consistency across representations is the evaluation metric
  repo: https://github.com/earmingol/cell2cell
- name: Ligand-Receptor Pairs curated list
  role: Source of the Human-2020-Jin ligand-receptor pairs used to compute communication
    scores for each tensor dimension
  repo: https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:ba7418470087814e3b1837914f6d0442bc9f7383f96fde95275f8384ab91e0bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - Ligand-Receptor Pairs curated list
  - Tensor-cell2cell
  - cell2cell
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# batch-effect-correction-combat-scanorama

## Summary

Applies ComBat and Scanorama batch-effect correction (via cell2cell.preprocessing) to single-cell expression matrices from the BALF COVID-19 dataset before building 4D-communication tensors, and evaluates how much the corrected representations preserve Tensor-cell2cell communication patterns versus raw counts.

## When to use

Use when building context-wise 4D-communication tensors from multi-donor single-cell data (e.g. 12 BALF COVID-19 patients) where batch effects across samples could confound LR communication scores, and you need batch-corrected representations before tensor decomposition.

## When NOT to use

- Single-context data with no multi-sample/batch structure — correction is unnecessary before a per-context communication analysis
- When the goal is to preserve absolute count-level signal rather than cross-sample patterns, since raw-count decompositions are demonstrably less consistent (0.77 vs 0.86)
- Cell types not shared across all samples, which are excluded from the tensor in this workflow

## Inputs

- BALF COVID-19 single-cell count matrix (63k immune and epithelial cells; 3 control / 3 moderate / 6 severe patients)
- Curated human ligand-receptor pair list (Jin et al. 2021; Human-2020-Jin-LR-pairs.csv)
- Per-sample context/donor metadata for batch assignment

## Outputs

- Five expression representations per context (raw counts, log(CPM+1), non-zero fraction, ComBat, Scanorama)
- Five 4D-communication tensors (contexts x LR pairs x senders x receivers)
- Pairwise factor-similarity matrix (1 - CorrIndex) written to corrindex_similarities.csv

## How to apply

Generate five expression representations per context with cell2cell.preprocessing: raw counts, log(CPM+1) normalization, fraction of non-zero cells, ComBat batch-corrected, and Scanorama batch-corrected/integrated matrices, using the 63k immune and epithelial BALF COVID-19 cells and the Human-2020-Jin ligand-receptor pair list. For each representation build the contexts x LR pairs x sender x receiver tensor, keeping only cell types shared across all samples, then decompose with Tensor-cell2cell tensor component analysis (tensorly backend; switch init='svd' to init='random' if memory errors arise). Compare all pairwise decompositions as 1 - CorrIndex with cell2cell.stats utilities: corrected/normalized strategies agree at mean similarity ~0.86, while raw counts diverge (mean similarity ~0.77 versus all other approaches), so raw counts should not be treated as equivalent to batch-corrected inputs.

## Related tools

- **cell2cell** (Provides cell2cell.preprocessing for generating raw, log(CPM+1), non-zero fraction, ComBat, and Scanorama representations, and cell2cell.stats for CorrIndex-based factor similarity comparisons) — https://github.com/earmingol/cell2cell
- **Tensor-cell2cell** (Decomposes each batch-corrected 4D-communication tensor into interpretable factors whose consistency across representations is the evaluation metric) — https://github.com/earmingol/cell2cell
- **Ligand-Receptor Pairs curated list** (Source of the Human-2020-Jin ligand-receptor pairs used to compute communication scores for each tensor dimension) — https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs

## Examples

```
pip install cell2cell  # then use cell2cell.preprocessing for ComBat/Scanorama correction and cell2cell.stats for 1-CorrIndex comparisons
```

## Evaluation signals

- Mean pairwise (1-CorrIndex) similarity across the five preprocessing strategies should approximate 0.86
- Mean similarity between the raw-counts decomposition and all other strategies should approximate 0.77
- corrindex_similarities.csv contains the full 5x5 pairwise similarity matrix with no missing pairs
- Each decomposition converges with the tensorly backend; if init='svd' raises memory errors, rerun with init='random' and confirm comparable factors
- All five tensors share the same dimensions restricted to cell types present in every sample

## Limitations

- The implemented tensor decomposition algorithm is not optimized for missing values, so only cell types shared across all samples are included, potentially missing rare cell types
- Raw counts yield systematically lower decomposition consistency (~0.77) than any normalized or batch-corrected representation, so results are not interchangeable across representations
- Reproducibility depends on exact versions of code and software as deposited in the Code Ocean capsule (https://doi.org/10.24433/CO.0051950.v2)

## Evidence

- [results] Tensor-cell2cell detects consistent cell-cell communication signatures across log(CPM+1), non-zero fraction, ComBat, and Scanorama preprocessing (mean similarity 0.86), with raw counts being the exception (mean similarity 0.77 versus all other approaches).: [evidence span withheld — anchor status: not_found]
- [methods] cell2cell.preprocessing includes functions for data preprocessing and integration: "cell2cell.preprocessing : Includes functions for manipulating data and variables (e.g. data preprocessing, integration, permutation, among others)."
- [readme] README points to the compendium of ligand-receptor pair lists used to source the human LR pairs: "This compendium has been created to collect lists of ligand-receptor pairs
and interactions for surface-secreted proteins that are available in literature."
- [discussion] Only cell types shared across all samples were included in the COVID-19 tensors: "we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases"
