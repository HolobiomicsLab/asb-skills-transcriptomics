---
name: ligand-receptor-interaction-scoring
description: Use when single-nucleus or single-cell RNA-seq data with cell-type annotations
  and a curated ligand-receptor pair list (e.g. Human-2020-Jin-LR-pairs.csv) must
  be converted into per-context communication scores for tensor decomposition.
when_to_use_negative:
- The goal is pathway-level rather than LR-pair-level scoring — CellChat generates
  scores at the signaling pathway level instead of LR pairs
- 'Cell types are not shared across all contexts: the implemented tensor workflow
  only retains cell types shared across all samples, so rare context-specific cell
  types would be dropped'
- The decomposition algorithm is not optimized for missing values, so tensors with
  many missing LR pairs or cell types per context are a poor fit
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0128
tools:
- name: cell2cell
  role: Computes ligand-receptor communication scores per context (preprocessing,
    non-zero fraction aggregation) and assembles them into the 3D/4D communication
    tensor via cell2cell.tensor
  repo: https://github.com/earmingol/cell2cell
- name: Tensor-cell2cell
  role: 'Downstream consumer: deconvolves the assembled 4D communication tensor into
    context-aware factors'
  repo: https://github.com/earmingol/cell2cell
- name: Ligand-Receptor Pairs curated list
  role: Source of the curated LR pair list (Human-2020-Jin-LR-pairs.csv) used as interaction
    prior
  repo: https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs
- name: CellChat
  role: 'Alternative score generator: its pathway-level or LR-pair output communication
    scores can be fed into Tensor-cell2cell for multi-context analysis'
  repo: https://github.com/sqjin/CellChat
- name: LIANA
  role: Alternative CCC tool whose output communication scores can be used to build
    the 4D tensor
  repo: https://github.com/saezlab/liana-py
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:e8897b34ccbe107d205bf1d4a1626ef8ea52cb5347c36c63ddc97d22925d3ca5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - Anaconda
  - CellChat
  - LIANA
  - Ligand-Receptor Pairs curated list
  - Tensor-cell2cell
  - cell2cell
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# ligand-receptor-interaction-scoring

## Summary

Computes cell-cell communication scores from the expression of each ligand and receptor in a curated LR pair list for every sender-receiver cell pair, yielding communication matrices per context that can be aggregated into 3D/4D tensors for Tensor-cell2cell factorization.

## When to use

Use when single-nucleus or single-cell RNA-seq data with cell-type annotations and a curated ligand-receptor pair list (e.g. Human-2020-Jin-LR-pairs.csv) must be converted into per-context communication scores for tensor decomposition.

## When NOT to use

- The goal is pathway-level rather than LR-pair-level scoring — CellChat generates scores at the signaling pathway level instead of LR pairs
- Cell types are not shared across all contexts: the implemented tensor workflow only retains cell types shared across all samples, so rare context-specific cell types would be dropped
- The decomposition algorithm is not optimized for missing values, so tensors with many missing LR pairs or cell types per context are a poor fit

## Inputs

- Single-cell / single-nucleus RNA-seq expression matrices per sample or context (e.g. 23-sample ASD prefrontal cortex dataset, PRJNA434002)
- Cell-type annotations per context (e.g. 16 shared sender and receiver cell types)
- Curated ligand-receptor pair list CSV (e.g. Human-2020-Jin-LR-pairs.csv, 749 LR pairs)

## Outputs

- Per-context communication matrices (LR pairs x sender cell types x receiver cell types)
- A 4D communication tensor object (contexts x LR pairs x senders x receivers), serializable as communication_tensor.pkl

## How to apply

Load single-cell expression data and cell-type annotations per context via cell2cell.datasets, and a curated LR pair list such as Human-2020-Jin-LR-pairs.csv from the LewisLabUCSD Ligand-Receptor-Pairs repository. For each LR pair, compute sender ligand and receiver receptor scores, taking the minimum across subunits for heteromeric ligands or receptors. Aggregate single-cell scores to cell-type level using the non-zero fraction strategy implemented in cell2cell.preprocessing, so sender/receiver communication scores reflect the fraction of cells expressing the ligand/receptor. Assemble per-context scores into a 3D communication matrix (LR pairs x sender cell types x receiver cell types) and stack contexts into a 4D tensor with cell2cell.tensor, retaining only cell types shared across all samples (e.g. 16 sender and 16 receiver types in the ASD study). Output communication matrices are ready for Tensor-cell2cell factorization or for use as scores from other tools such as CellChat or LIANA.

## Related tools

- **cell2cell** (Computes ligand-receptor communication scores per context (preprocessing, non-zero fraction aggregation) and assembles them into the 3D/4D communication tensor via cell2cell.tensor) — https://github.com/earmingol/cell2cell
- **Tensor-cell2cell** (Downstream consumer: deconvolves the assembled 4D communication tensor into context-aware factors) — https://github.com/earmingol/cell2cell
- **Ligand-Receptor Pairs curated list** (Source of the curated LR pair list (Human-2020-Jin-LR-pairs.csv) used as interaction prior) — https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs
- **CellChat** (Alternative score generator: its pathway-level or LR-pair output communication scores can be fed into Tensor-cell2cell for multi-context analysis) — https://github.com/sqjin/CellChat
- **LIANA** (Alternative CCC tool whose output communication scores can be used to build the 4D tensor) — https://github.com/saezlab/liana-py

## Examples

```
import cell2cell; scores = cell2cell.preprocessing.get_communication_scores(expression_data, lr_pairs=lr_pairs, communication_score='expression_fraction')
```

## Evaluation signals

- The assembled tensor is a valid 4D object whose dimensions correspond to contexts, LR pairs, sender cell types, and receiver cell types
- Reloading the serialized pickle (e.g. communication_tensor.pkl) reproduces the original communication scores
- Cell-type dimension contains only cell types shared across all samples (e.g. 16 senders and 16 receivers in the ASD case study)
- Heteromeric ligands/receptors receive the minimum score across their subunits, matching the documented scoring rule
- Cell-type-level scores reflect the non-zero fraction aggregation strategy rather than mean expression

## Limitations

- Only cell types shared across all samples can be included, potentially missing rare cell types
- The downstream tensor decomposition algorithm is not optimized for missing values in the communication tensor
- Benchmarking of LR-score-based CCC predictions is challenging due to the lack of a ground truth

## Evidence

- [other] Tensor-cell2cell computes communication scores from the expression of the ligand and the receptor in each LR pair for every sender-receiver cell pair, repeating the process for every LR pair to yield communication matrices that are aggregated into a 3D-communication tensor: [evidence span withheld — anchor status: not_found]
- [other] Single-cell expression is aggregated to the cell-type level using the non-zero fraction of cells, which is included as part of the Tensor-cell2cell built-in workflow.: [evidence span withheld — anchor status: not_found]
- [discussion] we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases: "we built a 4D-communication tensor that contains only the cell types that are shared across all samples"
- [results] we use a list of LR pairs published with CellChat (Jin et al. 2021, Nature Communications): "we use a list of LR pairs published with CellChat (Jin et al. 2021, Nature Communications)"
- [methods] Find a curated list of ligand-receptor pairs for your analyses at our GitHub Repository (https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs): "Find a curated list of ligand-receptor pairs for your analyses at our GitHub Repository"
