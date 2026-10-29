---
name: single-cell-to-cell-type-aggregation-nonzero-fraction
description: Use when raw single-cell expression data must be collapsed to cell-type-level
  sender/receiver scores per ligand-receptor pair before assembling communication
  tensors across multiple samples or contexts.
when_to_use_negative:
- Data are already aggregated to pseudo-bulk or cell-type-level expression, so a second
  single-cell aggregation step is unnecessary
- The analysis targets a single sample/context with no need to build a context-aware
  tensor; a single-context CCC tool alone may suffice
- Cell types are not shared across all samples and retaining rare context-specific
  cell types is a priority, since the standard workflow drops unshared cell types
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: cell2cell
  role: Provides the non-zero fraction aggregation strategy in cell2cell.preprocessing
    and the tensor module for assembling communication scores into 3D/4D tensors
  repo: https://github.com/earmingol/cell2cell
- name: Tensor-cell2cell
  role: Consumes the aggregated cell-type-level communication scores as the 4D tensor
    input for context-aware TCA decomposition
  repo: https://github.com/earmingol/cell2cell
- name: LewisLabUCSD/Ligand-Receptor-Pairs
  role: Source of the curated LR pair list (Human-2020-Jin-LR-pairs.csv) used to compute
    ligand and receptor scores prior to aggregation
  repo: https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:d981ef136a11df46a163c3cf37b9f9026b54eeb89def470b977e697162b1b57c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - LewisLabUCSD/Ligand-Receptor-Pairs
  - Tensor-cell2cell
  - cell2cell
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# single-cell-to-cell-type-aggregation-nonzero-fraction

## Summary

Aggregates single-cell ligand/receptor expression scores into cell-type-level communication scores using a non-zero fraction strategy implemented in cell2cell.preprocessing, as a prerequisite step for building 3D/4D communication tensors in the Tensor-cell2cell pipeline.

## When to use

Use when raw single-cell expression data must be collapsed to cell-type-level sender/receiver scores per ligand-receptor pair before assembling communication tensors across multiple samples or contexts.

## When NOT to use

- Data are already aggregated to pseudo-bulk or cell-type-level expression, so a second single-cell aggregation step is unnecessary
- The analysis targets a single sample/context with no need to build a context-aware tensor; a single-context CCC tool alone may suffice
- Cell types are not shared across all samples and retaining rare context-specific cell types is a priority, since the standard workflow drops unshared cell types

## Inputs

- single-cell expression matrices with cell-type annotations per context (loaded via cell2cell.datasets toy loaders or user data)
- curated ligand-receptor pair list (e.g., Human-2020-Jin-LR-pairs.csv from LewisLabUCSD/Ligand-Receptor-Pairs)

## Outputs

- cell-type-level communication scores per LR pair, sender and receiver cell type
- 3D communication tensor per context (LR pairs x senders x receivers) suitable for stacking into a 4D tensor

## How to apply

First compute, for each LR pair (e.g., from Human-2020-Jin-LR-pairs.csv), the sender ligand and receiver receptor scores from the single-cell expression matrix, taking the minimum across subunits for heteromeric ligands or receptors. Then apply the non-zero fraction aggregation strategy from cell2cell.preprocessing to collapse single-cell scores into cell-type-level communication scores. This aggregation is built into the Tensor-cell2cell workflow, so it can be invoked as part of the standard preprocessing before communication matrices are assembled into a per-context 3D tensor (LR pairs x sender cell types x receiver cell types) and stacked into a 4D tensor. Retain only cell types shared across all samples when stacking contexts, as done in the COVID-19 and ASD case-study tensors.

## Related tools

- **cell2cell** (Provides the non-zero fraction aggregation strategy in cell2cell.preprocessing and the tensor module for assembling communication scores into 3D/4D tensors) — https://github.com/earmingol/cell2cell
- **Tensor-cell2cell** (Consumes the aggregated cell-type-level communication scores as the 4D tensor input for context-aware TCA decomposition) — https://github.com/earmingol/cell2cell
- **LewisLabUCSD/Ligand-Receptor-Pairs** (Source of the curated LR pair list (Human-2020-Jin-LR-pairs.csv) used to compute ligand and receptor scores prior to aggregation) — https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs

## Examples

```
from cell2cell import preprocessing; agg_scores = preprocessing.aggregate_single_cell_scores(single_cell_data, labels, strategy='non_zero_fraction')
```

## Evaluation signals

- The aggregated scores are cell-type level: each entry corresponds to one sender cell type and one receiver cell type for a given LR pair, not to individual cells
- The resulting communication matrices assemble into a valid 3D tensor per context with dimensions LR pairs x sender cell types x receiver cell types, and stacked contexts yield a 4D tensor
- Only cell types shared across all samples appear in the tensor dimensions
- Serializing the tensor (e.g., communication_tensor.pkl) and reloading reproduces identical communication scores
- Heteromeric ligand/receptor scores reflect the minimum across subunits before aggregation

## Limitations

- The standard workflow retains only cell types shared across all samples, potentially missing rare cell types present in only some contexts
- The implemented tensor decomposition algorithm is not optimized for missing values, limiting inclusion of LR pairs or cell types absent in some contexts
- Benchmarking aggregated communication predictions is challenging due to the lack of a ground truth

## Evidence

- [results] non-zero fraction aggregation: [evidence span withheld — anchor status: not_found]
- [methods] heteromeric minimum rule: [evidence span withheld — anchor status: not_found]
- [results] 3D to 4D tensor assembly: [evidence span withheld — anchor status: not_found]
- [discussion] shared cell types only: "we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases"
- [readme] LR pair source README: [evidence span withheld — anchor status: external_doc]
