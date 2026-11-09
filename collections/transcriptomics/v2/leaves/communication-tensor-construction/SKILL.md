---
name: communication-tensor-construction
description: Use when multiple single-cell samples/contexts must be converted into
  a single 4D communication tensor ready for Tensor-cell2cell tensor decomposition.
when_to_use_negative:
- Only one sample/context is available — a single context does not form a 4D tensor
  and pairwise per-context CCC tools suffice
- The analysis requires including LR pairs or cell types missing in some contexts
  — the algorithm is not optimized for missing values, so shared cell types only are
  retained
- The input is already a communication-score matrix or pre-built tensor; in that case
  proceed directly to Tensor-cell2cell factorization
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: cell2cell
  role: Provides preprocessing (aggregation to non-zero fraction, LR score computation)
    and tensor-stacking modules used to build the 4D tensor
  repo: https://github.com/earmingol/cell2cell
- name: Tensor-cell2cell
  role: Downstream consumer of the constructed 4D tensor; validates that tensor dimensions
    match its decomposition input expectations
  repo: https://github.com/earmingol/cell2cell
- name: Ligand-Receptor-Pairs
  role: Source of the curated LR pair list (Human-2020-Jin-LR-pairs.csv) used as the
    LR-pair dimension
  repo: https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs
provenance:
  source_task_ids:
  - task_007
  - task_004
  - task_008
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:d8a23e77573dfde7ba865742215b32da2ce34c24678b34ae78cd57a86ff71a38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - Anaconda
  - CellChat
  - Ligand-Receptor-Pairs
  - Tensor-cell2cell
  - cell2cell
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# communication-tensor-construction

## Summary

Constructs a 4D cell-cell communication tensor (contexts x ligand-receptor pairs x sender cell types x receiver cell types) by computing per-LR-pair communication scores from single-cell expression and stacking per-context 3D tensors, as implemented in the cell2cell preprocessing and tensor modules.

## When to use

Use when multiple single-cell samples/contexts must be converted into a single 4D communication tensor ready for Tensor-cell2cell tensor decomposition.

## When NOT to use

- Only one sample/context is available — a single context does not form a 4D tensor and pairwise per-context CCC tools suffice
- The analysis requires including LR pairs or cell types missing in some contexts — the algorithm is not optimized for missing values, so shared cell types only are retained
- The input is already a communication-score matrix or pre-built tensor; in that case proceed directly to Tensor-cell2cell factorization

## Inputs

- Single-cell expression matrices with cell-type annotations for each context/sample
- Curated ligand-receptor pair list (e.g., Human-2020-Jin-LR-pairs.csv, Jin et al. 2021)
- Context/sample metadata (e.g., control, moderate, severe patient labels)

## Outputs

- 4D-communication tensor (contexts x LR pairs x sender cell types x receiver cell types), e.g., serialized as communication_tensor.pkl
- Per-context 3D communication score tensors (LR pairs x sender x receiver)

## How to apply

Load each context's single-cell expression matrix and cell-type annotations via cell2cell, and a curated LR pair list such as Human-2020-Jin-LR-pairs.csv from the LewisLabUCSD repository. For each LR pair, compute sender ligand and receiver receptor scores, taking the minimum across subunits for heteromeric ligands or receptors, then aggregate single-cell scores to cell-type level using the non-zero fraction strategy in cell2cell.preprocessing. Assemble each context's scores into a 3D tensor (LR pairs x sender x receiver) and stack all contexts into a 4D tensor with cell2cell.tensor, retaining only cell types shared across all samples. Serialize the object (e.g., communication_tensor.pkl) so the decomposition can reload identical scores; dimensional consistency across contexts is the key decision point before factorization.

## Related tools

- **cell2cell** (Provides preprocessing (aggregation to non-zero fraction, LR score computation) and tensor-stacking modules used to build the 4D tensor) — https://github.com/earmingol/cell2cell
- **Tensor-cell2cell** (Downstream consumer of the constructed 4D tensor; validates that tensor dimensions match its decomposition input expectations) — https://github.com/earmingol/cell2cell
- **Ligand-Receptor-Pairs** (Source of the curated LR pair list (Human-2020-Jin-LR-pairs.csv) used as the LR-pair dimension) — https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs

## Examples

```
from cell2cell.tensor import preprocessing; tensor = preprocessing.get_tensor_from_data(rnaseq_data, lr_pairs, how='outer', communication_score='expression_fraction'); tensor.pickle_load('communication_tensor.pkl')
```

## Evaluation signals

- Reloaded pickle reproduces the original communication scores element-wise
- Tensor has exactly 4 dimensions corresponding to contexts, LR pairs, sender cell types, and receiver cell types
- Only cell types shared across all samples are present in the sender/receiver dimensions (e.g., 16 sender and 16 receiver cell types in the ASD 23x749x16x16 tensor)
- Heteromeric ligand/receptor subunits resolved by minimum across subunits before aggregation
- Cell-type-level scores derive from the non-zero fraction aggregation of single-cell expression

## Limitations

- Only cell types shared across all samples were included in the COVID-19 and ASD case-study tensors, potentially missing rare cell types
- The implemented tensor decomposition algorithm is not optimized for missing values, so contexts lacking cell types or LR pairs cannot be included

## Evidence

- [results] Single-cell expression is aggregated to the cell-type level using the non-zero fraction of cells, which is included as part of the Tensor-cell2cell built-in workflow.: [evidence span withheld — anchor status: not_found]
- [discussion] we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases: "we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases"
- [methods] cell2cell.preprocessing : Includes functions for manipulating data and variables (e.g. data preprocessing, integration, permutation, among others).: "cell2cell.preprocessing : Includes functions for manipulating data and variables (e.g. data preprocessing, integration, permutation, among others)."
- [readme] Find a curated list of ligand-receptor pairs for your analyses at our GitHub Repository: "Find a curated list of ligand-receptor pairs for your analyses at our GitHub Repository"
