---
name: tensor-factorization-of-communication-scores
description: Use when you have per-context communication scores (e.g., CellChat LR-pair
  scores for control/moderate/severe COVID-19 BALF samples) and need to deconvolve
  them into factors that vary jointly across LR pairs, cell types, and patient contexts.
when_to_use_negative:
- You only need communication inference in a single sample or context — use a standard
  CCC tool (e.g., CellChat) rather than tensor decomposition
- Your communication tensor contains many missing LR pairs or cell types per context;
  the implemented algorithm is not optimized for missing values
- You aim to include cell types not shared across all samples — the case-study tensors
  only contain cell types shared across all samples
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Tensor-cell2cell
  role: Performs the non-negative TCA of the 4D-communication tensor into 10 interpretable
    factors
  repo: https://github.com/earmingol/cell2cell
- name: cell2cell
  role: Python suite providing the cell2cell.tensor module for tensor decomposition
    and cell2cell.stats for Gini-coefficient statistics of factor loadings
  repo: https://github.com/earmingol/cell2cell
- name: CellChat
  role: Computes per-context ligand-receptor communication scores (Human-2020-Jin
    pair list) that populate the tensor
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:17877ddc4e250389a389bb974d017a9296fd8a3a11373ae4c8297d8a50394c4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - CellChat
  - Tensor-cell2cell
  - cell2cell
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# tensor-factorization-of-communication-scores

## Summary

Decompose a 4D ligand-receptor x sender x receiver x context communication tensor (e.g., the 12 x 189 x 6 x 6 BALF COVID-19 tensor built from CellChat Human-2020-Jin scores) with Tensor-cell2cell at a fixed rank (e.g., rank=10, non-negative, init='svd') to extract context-dependent communication factors whose loadings correlate with phenotypic severity.

## When to use

Use when you have per-context communication scores (e.g., CellChat LR-pair scores for control/moderate/severe COVID-19 BALF samples) and need to deconvolve them into factors that vary jointly across LR pairs, cell types, and patient contexts.

## When NOT to use

- You only need communication inference in a single sample or context — use a standard CCC tool (e.g., CellChat) rather than tensor decomposition
- Your communication tensor contains many missing LR pairs or cell types per context; the implemented algorithm is not optimized for missing values
- You aim to include cell types not shared across all samples — the case-study tensors only contain cell types shared across all samples

## Inputs

- Prebuilt 4D-communication tensor (12 contexts x 189 LR pairs x 6 senders x 6 receivers) from GSE145926 BALF COVID-19 samples
- Per-context cell-cell communication scores computed with CellChat and the Human-2020-Jin-LR-pairs.csv ligand-receptor list
- Context/patient severity labels (control, moderate, severe) for correlating factor loadings

## Outputs

- Ten factor decompositions with LR-pair, sender-cell, receiver-cell, and context loadings
- Spearman rank correlations of factor loadings against a reference decomposition
- Gini coefficients of factor loadings computed with cell2cell.stats
- Ranked list of top ligand-receptor pairs per factor (tensor_factorization_results.csv)

## How to apply

Build a 4D-communication tensor (contexts x LR-pairs x sender cells x receiver cells) from communication scores computed per context with a scoring tool such as CellChat using the Human-2020-Jin ligand-receptor pair list (GSE145926 BALF data: three control, three moderate, six severe patients). Run non-negative tensor factorization with Tensor-cell2cell (cell2cell.tensor module) at rank=10 with init='svd', switching to init='random' if memory errors occur. Extract LR-pair and context loadings per factor, compute Spearman rank correlations of factor loadings against a reference decomposition (expected ~0.72, 0.61, 0.92 for factors 1, 2, and 8), and compute the Gini coefficient of factor loadings with cell2cell.stats (factor 9 expected ~0.09) to quantify factor specificity. Rank LR pairs by factor loadings to identify severity-associated communication, e.g., APP/MDK/MIF/CD99, CCL2/CCL3/CCL8 with CCR1/CCR2/CCR5, and MIF-CD74/CD44.

## Related tools

- **Tensor-cell2cell** (Performs the non-negative TCA of the 4D-communication tensor into 10 interpretable factors) — https://github.com/earmingol/cell2cell
- **cell2cell** (Python suite providing the cell2cell.tensor module for tensor decomposition and cell2cell.stats for Gini-coefficient statistics of factor loadings) — https://github.com/earmingol/cell2cell
- **CellChat** (Computes per-context ligand-receptor communication scores (Human-2020-Jin pair list) that populate the tensor) — https://github.com/sqjin/CellChat

## Examples

```
pip install cell2cell
```

## Evaluation signals

- Spearman correlations of factors 1, 2, and 8 loadings against the reference decomposition fall within +/-0.05 of 0.72, 0.61, and 0.92 respectively
- Factor-9 Gini coefficient is within +/-0.02 of 0.09
- Top-ranked LR pairs in relevant factors include APP/MDK/MIF/CD99, CCL2/CCL3/CCL8 with CCR1/CCR2/CCR5, and MIF-CD74/CD44
- Factor 8 context loadings show the highest Spearman correlation (0.92) with COVID-19 severity
- Tensor shape matches 12 x 189 x 6 x 6 (contexts x LR pairs x senders x receivers) from GSE145926

## Limitations

- The implemented tensor decomposition algorithm is not optimized for missing values in the communication tensor, so LR pairs or cell types absent in some contexts cannot be fully included
- The COVID-19 case-study tensor contains only cell types shared across all samples, potentially missing rare cell types
- Non-negative factorization with init='svd' can raise memory errors, requiring a switch to init='random'

## Evidence

- [abstract] Here we present Tensor-cell2cell, an unsupervised method using tensor decomposition, which deciphers context-driven intercellular communication: "Here we present Tensor-cell2cell, an unsupervised method using tensor decomposition, which deciphers context-driven intercellular communication"
- [methods] deconvolving complex patterns of cell-cell communication across multiple samples or contexts simultaneously into interpretable factors: "deconvolving complex patterns of cell-cell communication across multiple samples or contexts simultaneously into interpretable factors"
- [results] a lung dataset of 63k immune and epithelial cells across three control, three moderate, and six severe COVID-19 patients: "a lung [dataset](https://doi.org/10.1038/s41591-020-0901-9) of 63k immune and epithelial cells across three control, three moderate, and six severe COVID-19 patients"
- [discussion] we built a 4D-communication tensor that contains only the cell types that are shared across all samples: "we built a 4D-communication tensor that contains only the cell types that are shared across all samples in our COVID-19 and ASD study cases"
- [readme] Step 3: Install cell2cell pip install cell2cell: [evidence span withheld — anchor status: not_found]
