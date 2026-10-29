---
name: rank-selection-by-reconstruction-error
description: Use when a 4D-communication tensor (cell types x LR pairs x contexts
  x cell types) has been built with cell2cell and the number of latent communication
  factors must be chosen before running non-negative tensor component analysis.
when_to_use_negative:
- The tensor contains missing values across contexts — the article notes the implemented
  tensor decomposition algorithm is not optimized for missing values, so rank selection
  on an incomplete tensor may be biased
- Communication scores have not yet been computed per context; rank selection requires
  an assembled 4D tensor, not raw single-cell expression data
- A single-context cell-cell communication analysis is the goal — this skill applies
  to multi-context (e.g., 12 time-point) tensors, not one-sample deconvolution
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Tensor-cell2cell
  role: Runs the non-negative tensor component analysis and rank scan whose reconstruction
    errors drive factor-number selection
  repo: https://github.com/earmingol/cell2cell
- name: cell2cell
  role: Provides the tensor module to assemble the 4D-communication tensor and the
    analysis pipelines for downstream factor extraction
  repo: https://github.com/earmingol/cell2cell
- name: tensorly
  role: Backend on which Tensor-cell2cell's tensor decomposition is built
- name: Ligand-Receptor-Pairs
  role: Curated LR-pair list (e.g., Human-2020-Jin-LR-pairs.csv) from which the LR-pair
    axis of the tensor is drawn
  repo: https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:5dc0db9a74f159cb8bf3643c4458ef7a02089b0589a733c1065c68ca4c38cb4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - Ligand-Receptor-Pairs
  - Tensor-cell2cell
  - cell2cell
  - tensorly
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# rank-selection-by-reconstruction-error

## Summary

Select the number of tensor factors for Tensor-cell2cell decomposition by scanning component ranks and choosing the rank that minimizes the normalized reconstruction error of the 4D-communication tensor.

## When to use

Use when a 4D-communication tensor (cell types x LR pairs x contexts x cell types) has been built with cell2cell and the number of latent communication factors must be chosen before running non-negative tensor component analysis.

## When NOT to use

- The tensor contains missing values across contexts — the article notes the implemented tensor decomposition algorithm is not optimized for missing values, so rank selection on an incomplete tensor may be biased
- Communication scores have not yet been computed per context; rank selection requires an assembled 4D tensor, not raw single-cell expression data
- A single-context cell-cell communication analysis is the goal — this skill applies to multi-context (e.g., 12 time-point) tensors, not one-sample deconvolution

## Inputs

- 4D-communication tensor (dimensions: sender cell types x LR pairs x contexts/time points x receiver cell types) built with cell2cell's tensor module
- Optional: noise-perturbed copy of the tensor (Gaussian noise added to communication scores) for robustness checking

## Outputs

- Selected number of factors (e.g., rank = 4) for the final Tensor-cell2cell decomposition
- Rank-selection curve of normalized reconstruction error per candidate rank
- Noise-robustness metrics confirming stability of the chosen rank
- Factor loadings for the context, LR-pair and cell-type axes at the selected rank (e.g., saved to simulation_tca_results.pkl)

## How to apply

Assemble the communication scores into the 4D tensor using cell2cell's tensor module, then run non-negative TCA via Tensor-cell2cell (built on tensorly) while scanning a range of component ranks. For each candidate rank, record the normalized reconstruction error between the input tensor and its rank-r factorization. Select the rank at which error is minimized or reaches an elbow — in the simulated 3-cell-type, 300-LR-pair, 12-time-point benchmark, four factors gave the decomposition that best minimized error, consistent with the four embedded patterns (oscillation, pulse, exponential decay, linear decrease). Repeat the scan with Gaussian noise added to the scores to confirm the selected rank is robust; then extract the context, LR-pair and cell-type factor loadings at the chosen rank and save loadings, rank-selection errors and noise-robustness metrics for downstream verification.

## Related tools

- **Tensor-cell2cell** (Runs the non-negative tensor component analysis and rank scan whose reconstruction errors drive factor-number selection) — https://github.com/earmingol/cell2cell
- **cell2cell** (Provides the tensor module to assemble the 4D-communication tensor and the analysis pipelines for downstream factor extraction) — https://github.com/earmingol/cell2cell
- **tensorly** (Backend on which Tensor-cell2cell's tensor decomposition is built)
- **Ligand-Receptor-Pairs** (Curated LR-pair list (e.g., Human-2020-Jin-LR-pairs.csv) from which the LR-pair axis of the tensor is drawn) — https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs

## Examples

```
import cell2cell; tensor = cell2cell.tensor.BaseTensor(communication_tensor); tensor.compute_tensor_factorization(rank=4, init='svd')
```

## Evaluation signals

- The rank-selection curve exhibits a minimum (or clear elbow) at the expected rank; in the simulated benchmark the minimum occurs at rank = 4, matching the four embedded communication patterns
- Factor loadings on the context axis quantitatively correspond to the four assigned temporal patterns: oscillation, pulse, exponential decay and linear decrease
- Re-running the rank scan on Gaussian-noise-perturbed scores reproduces the same rank-4 optimum, indicating robustness
- Decomposition is run in non-negative mode and normalized reconstruction error decreases monotonically with rank before the selected optimum is chosen on parsimony/error trade-off

## Limitations

- The implemented tensor decomposition algorithm is not optimized for missing values, so tensors with absent cell types or LR pairs in some contexts may bias rank selection
- Only cell types shared across all samples can be included, potentially missing rare cell types and distorting the rank estimate
- Benchmarking is hampered by the lack of ground truth, so rank choice must be justified by reconstruction error and pattern correspondence rather than external validation

## Evidence

- [other] Run non-negative tensor component analysis via Tensor-cell2cell, scanning component ranks and selecting the number of factors that minimizes the normalized reconstruction error (expected rank = 4): [evidence span withheld — anchor status: not_found]
- [other] In the simulated 4D-communication tensor, four factors gave the decomposition that best minimized error: [evidence span withheld — anchor status: not_found]
- [intro] Here, we introduce Tensor-cell2cell, a TCA-based strategy that deconvolves intercellular communication across multiple contexts: "Here, we introduce Tensor-cell2cell, a TCA-based strategy that deconvolves intercellular communication across multiple contexts and uncovers modules"
- [other] Repeat the decomposition with Gaussian noise added to the simulated scores to evaluate robustness of the rank-4 solution: [evidence span withheld — anchor status: not_found]
- [discussion] The implemented algorithm is not optimized for missing values in the communication tensor: "the implemented tensor decomposition algorithm can be further optimized for missing values. Since the implemented algorithm is not optimized for this purpose"
