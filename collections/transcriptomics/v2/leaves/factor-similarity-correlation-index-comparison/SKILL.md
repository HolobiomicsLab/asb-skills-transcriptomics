---
name: factor-similarity-correlation-index-comparison
description: Use when two or more 4D-communication tensors (contexts x LR pairs x
  senders x receivers) have been decomposed under different preprocessing, batch-correction,
  or parameter settings, and you need to test whether recovered communication factors
  are consistent.
when_to_use_negative:
- Only a single tensor decomposition is available and there is nothing to compare
  it against
- You need to compare raw communication scores (LR-pair score matrices) rather than
  decomposed factor structure; the article discusses how comparing CCC tools directly
  is hard because of qualitative differences in their outputs
- The tensors contain missing communication members (cell types or LR pairs absent
  from some contexts), since the implemented tensor decomposition is not optimized
  for missing values
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Tensor-cell2cell
  role: Performs the tensor component analysis whose resulting factors are compared
    across decompositions via 1 - CorrIndex
  repo: https://github.com/earmingol/cell2cell
- name: cell2cell
  role: Provides cell2cell.stats comparison utilities and cell2cell.preprocessing
    functions (normalization, ComBat/Scanorama integration) used to generate the variant
    tensors
  repo: https://github.com/earmingol/cell2cell
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:80917a98187e4029cbccf7bcdfe4e1887489b020a592fa13c61d12adec1e1ba3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - Tensor-cell2cell
  - cell2cell
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# factor-similarity-correlation-index-comparison

## Summary

Quantifies consistency between Tensor-cell2cell tensor decompositions (e.g., built from different preprocessing/batch-correction strategies) by computing pairwise factor similarities as 1 - CorrIndex using cell2cell.stats utilities. In the BALF COVID-19 case, this yields mean similarity ~0.86 across log(CPM+1), non-zero fraction, ComBat, and Scanorama representations, with raw counts diverging (~0.77).

## When to use

Use when two or more 4D-communication tensors (contexts x LR pairs x senders x receivers) have been decomposed under different preprocessing, batch-correction, or parameter settings, and you need to test whether recovered communication factors are consistent.

## When NOT to use

- Only a single tensor decomposition is available and there is nothing to compare it against
- You need to compare raw communication scores (LR-pair score matrices) rather than decomposed factor structure; the article discusses how comparing CCC tools directly is hard because of qualitative differences in their outputs
- The tensors contain missing communication members (cell types or LR pairs absent from some contexts), since the implemented tensor decomposition is not optimized for missing values

## Inputs

- Factor loadings from Tensor-cell2cell (tensorly backend) TCA decompositions of 4D-communication tensors (contexts x LR pairs x sender cell types x receiver cell types)
- Multiple tensor representations per preprocessing variant: raw counts, log(CPM+1), fraction of non-zero cells, ComBat-corrected, Scanorama-corrected/integrated

## Outputs

- Pairwise factor-similarity matrix of 1 - CorrIndex values between all decompositions
- corrindex_similarities.csv containing the full pairwise similarity matrix
- Summary statistics: mean 1-CorrIndex across all strategies and mean similarity of a divergent variant (raw counts) versus the rest

## How to apply

Decompose each 4D-communication tensor with Tensor-cell2cell tensor component analysis using the tensorly backend; if init='svd' raises memory errors, switch to init='random'. For every pair of decompositions, compute all pairwise factor similarities as 1 - CorrIndex with the cell2cell.stats comparison utilities, so values near 1 indicate matching factor subspace structure. Assemble the full 5x5 (or k x k) symmetric similarity matrix and summarize the mean across preprocessing strategies and between a divergent variant (e.g., raw counts) and all others. In the paper's BALF COVID-19 workflow (3 control / 3 moderate / 6 severe patients, 63k cells, Jin et al. 2021 human LR pairs), the expected mean pairwise 1-CorrIndex across log(CPM+1), non-zero fraction, ComBat, and Scanorama is approximately 0.86, while raw counts average approximately 0.77 against the other strategies. Write the full pairwise matrix to corrindex_similarities.csv for downstream inspection.

## Related tools

- **Tensor-cell2cell** (Performs the tensor component analysis whose resulting factors are compared across decompositions via 1 - CorrIndex) — https://github.com/earmingol/cell2cell
- **cell2cell** (Provides cell2cell.stats comparison utilities and cell2cell.preprocessing functions (normalization, ComBat/Scanorama integration) used to generate the variant tensors) — https://github.com/earmingol/cell2cell

## Examples

```
conda create -n cell2cell -y python=3.7 jupyter; conda activate cell2cell; pip install cell2cell
```

## Evaluation signals

- Mean pairwise 1-CorrIndex across the five preprocessing strategies (raw counts, log(CPM+1), non-zero fraction, ComBat, Scanorama) is approximately 0.86, matching the paper
- Mean 1-CorrIndex between the raw-counts decomposition and all other strategies is approximately 0.77, matching the paper
- The pairwise similarity matrix is symmetric with ones on the diagonal and all values in [0, 1]
- Decompositions reproduce whether run with init='svd' or init='random' (random is the fallback when svd raises memory errors)
- corrindex_similarities.csv is written with a complete k x k comparison (no missing pairwise entries)

## Limitations

- Raw-count representations yield less consistent decompositions (mean ~0.77 versus ~0.86 for other strategies), so raw counts are the exception to expected consistency
- init='svd' initialization can trigger memory errors on large tensors, requiring a switch to init='random'
- Only cell types shared across all samples were included in the COVID-19 tensors, potentially missing rare cell types, and the decomposition algorithm is not optimized for missing values
- Reproducibility depends on the exact versions of code and software deposited in the Code Ocean capsule (https://doi.org/10.24433/CO.0051950.v2)

## Evidence

- [abstract] Here we present Tensor-cell2cell, an unsupervised method using tensor decomposition, which deciphers context-driven intercellular communication: "Here we present Tensor-cell2cell, an unsupervised method using tensor decomposition, which deciphers context-driven intercellular communication"
- [discussion] Tensor-cell2cell results can be coupled with downstream analysis methods to facilitate interpretation: "Tensor-cell2cell results can be coupled with downstream analysis methods to facilitate interpretation and provide further insights of underlying mechanisms."
- [methods] cell2cell.preprocessing includes functions for data preprocessing and integration: "cell2cell.preprocessing : Includes functions for manipulating data and variables (e.g. data preprocessing, integration, permutation, among others)."
- [discussion] all our analyses including data and exact versions of code and software were deposited in a Code Ocean capsule: "we deposited all our analyses including data and exact versions of code and software in a Code Ocean capsule"
- [readme] Step 3: Install cell2cell via pip: [evidence span withheld — anchor status: not_found]
