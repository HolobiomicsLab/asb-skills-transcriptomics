---
name: tensor-factorization-decomposition
description: 'Use when you have communication scores from multiple contexts (e.g.,
  12 BALF COVID-19 samples: 3 control, 3 moderate, 6 severe) and need to deconvolve
  them into context-driven communication patterns simultaneously.'
when_to_use_negative:
- When analyzing a single sample or context — use regular cell2cell for single-context
  CCC inference instead of tensor decomposition
- When the communication tensor contains many missing values — the implemented decomposition
  algorithm is not optimized for missing values, so only shared cell types and LR
  pairs should be included and members missing in some contexts are dropped
- When using raw (unnormalized) counts as the expression representation — they yield
  less consistent decompositions (mean 1-CorrIndex ≈ 0.77 vs ≈ 0.86 for log(CPM+1),
  non-zero fraction, ComBat, Scanorama)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Tensor-cell2cell
  role: Performs the non-negative tensor component analysis that deconvolves the 4D-communication
    tensor into interpretable context-dependent factors
  repo: https://github.com/earmingol/cell2cell
- name: cell2cell
  role: Python suite providing tensor construction (cell2cell.tensor), preprocessing
    (cell2cell.preprocessing), and factor-comparison/statistics utilities for computing
    CorrIndex similarities
  repo: https://github.com/earmingol/cell2cell
- name: LIANA
  role: Runs CellChat, CellPhoneDB, NATMI, and SingleCellSignalR scoring per context
    to generate the communication scores used as tensor input
  repo: https://github.com/saezlab/liana-py
- name: CellChat
  role: Source of the Human-2020-Jin LR database and one of the communication scoring
    methods whose outputs can feed the tensor
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:928c69db03d61299a5c88cf6047d0f3a7de4c737fc382856709cd57f34c2612a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - CellChat
  - LIANA
  - Tensor-cell2cell
  - cell2cell
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# tensor-factorization-decomposition

## Summary

Decompose 4D cell-cell communication tensors (LR pairs × sender cell types × receiver cell types × contexts) with non-negative Tensor Component Analysis (TCA) in Tensor-cell2cell to extract latent, context-dependent communication modules as interpretable factors.

## When to use

Use when you have communication scores from multiple contexts (e.g., 12 BALF COVID-19 samples: 3 control, 3 moderate, 6 severe) and need to deconvolve them into context-driven communication patterns simultaneously.

## When NOT to use

- When analyzing a single sample or context — use regular cell2cell for single-context CCC inference instead of tensor decomposition
- When the communication tensor contains many missing values — the implemented decomposition algorithm is not optimized for missing values, so only shared cell types and LR pairs should be included and members missing in some contexts are dropped
- When using raw (unnormalized) counts as the expression representation — they yield less consistent decompositions (mean 1-CorrIndex ≈ 0.77 vs ≈ 0.86 for log(CPM+1), non-zero fraction, ComBat, Scanorama)

## Inputs

- Per-context cell-cell communication score matrices (e.g., from CellChat, CellPhoneDB, NATMI, SingleCellSignalR via LIANA, or Tensor-cell2cell's built-in rank-based scores)
- Preprocessed single-cell RNA-seq counts (e.g., BALF COVID-19 dataset: 63k immune and epithelial cells across 3 control, 3 moderate, 6 severe patients)
- Curated ligand-receptor pair list (e.g., Human-2020-Jin-LR-pairs.csv, Jin et al. 2021)

## Outputs

- Decomposed tensor factors representing communication modules (cell-type, LR-pair, and context loadings per rank)
- Pairwise decomposition similarity matrix as 1-CorrIndex between factorizations (e.g., tensor_similarity_scores.csv, 5×5 matrix with mean ~0.82)
- Interpretable factor-specific communication networks for downstream analyses (GSEA, clustering, visualization, statistical comparison)

## How to apply

Build a 4D-communication tensor (contexts × LR pairs × sender cell types × receiver cell types) from per-context communication scores (e.g., CellChat, CellPhoneDB, NATMI, SingleCellSignalR via LIANA, or cell2cell's built-in rank-based scores), retaining only LR pairs and cell types shared across all samples (e.g., 172 shared LR pairs for the BALF COVID-19 study). Decompose with non-negative tensor component analysis using the tensorly backend; if memory errors arise with init='svd', switch to init='random' per the documentation. Assess decomposition consistency across alternative inputs by computing pairwise 1-CorrIndex similarities between factorizations (e.g., mean 0.82 across five scoring methods; 0.86 across preprocessing strategies, with raw counts an outlier at 0.77). This joint decomposition preserves the correlation structure across contexts that per-context analyses lose.

## Related tools

- **Tensor-cell2cell** (Performs the non-negative tensor component analysis that deconvolves the 4D-communication tensor into interpretable context-dependent factors) — https://github.com/earmingol/cell2cell
- **cell2cell** (Python suite providing tensor construction (cell2cell.tensor), preprocessing (cell2cell.preprocessing), and factor-comparison/statistics utilities for computing CorrIndex similarities) — https://github.com/earmingol/cell2cell
- **LIANA** (Runs CellChat, CellPhoneDB, NATMI, and SingleCellSignalR scoring per context to generate the communication scores used as tensor input) — https://github.com/saezlab/liana-py
- **CellChat** (Source of the Human-2020-Jin LR database and one of the communication scoring methods whose outputs can feed the tensor) — https://github.com/sqjin/CellChat

## Examples

```
import cell2cell; tensor = cell2cell.tensor.BaseTensor(); tensor.compute_tensor_factorization(rank=ranks, init='random')
```

## Evaluation signals

- Mean pairwise 1-CorrIndex similarity across decompositions of the five scoring methods on BALF COVID-19 samples should be ≈0.82
- Mean pairwise 1-CorrIndex across preprocessing strategies (log(CPM+1), non-zero fraction, ComBat, Scanorama) should be ≈0.86; raw counts vs others ≈0.77
- The pairwise similarity output is a complete 5×5 matrix (e.g., tensor_similarity_scores.csv) with off-diagonal values in [0, 1]
- Tensor dimensions must match the retained shared members: contexts × LR pairs (e.g., 172 shared LR pairs) × sender cell types × receiver cell types common to all samples
- Decomposition should complete without memory errors; if init='svd' fails, rerunning with init='random' should still converge to a consistent factorization

## Limitations

- The implemented tensor decomposition algorithm is not optimized for missing values, so only cell types and LR pairs shared across all samples can be included, potentially missing rare cell types
- Further developments are needed to include all possible communication members (LR pairs and cell types missing in some contexts)
- Reproducibility depends on the exact data, code, and software versions deposited in the Code Ocean capsule (https://doi.org/10.24433/CO.0051950.v2)

## Evidence

- [abstract] Here we present Tensor-cell2cell, an unsupervised method using tensor decomposition, which deciphers context-driven intercellular communication: "Here we present Tensor-cell2cell, an unsupervised method using tensor decomposition, which deciphers context-driven intercellular communication"
- [discussion] we built a 4D-communication tensor that contains only the cell types that are shared across all samples: "we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases"
- [results] if memory errors arise with init='svd', switch to init='random': [evidence span withheld — anchor status: not_found]
- [methods] Reproducible runs of the analyses in the Tensor-cell2cell paper are available at CodeOcean.com: [evidence span withheld — anchor status: not_found]
- [intro] Existing methods lose the correlation structure across contexts: "current approaches cannot account for high variability in contexts (e.g., multiple time points or phenotypic states) simultaneously. Existing methods lose the correlation structure across contexts"
