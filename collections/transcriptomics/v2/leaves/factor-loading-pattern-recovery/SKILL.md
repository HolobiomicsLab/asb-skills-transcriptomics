---
name: factor-loading-pattern-recovery
description: Use when a simulated or real 4D-communication tensor has known embedded
  latent patterns and you need to verify that tensor decomposition recovers them from
  context, LR-pair, and cell-type factor loadings.
when_to_use_negative:
- The tensor decomposition algorithm is not optimized for missing values, so tensors
  with missing cell types or LR pairs in some contexts are the wrong input for this
  skill
- Only cell types shared across all contexts can be included; if rare cell types absent
  from some samples are the analysis target, this skill will exclude them
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_2258
tools:
- name: Tensor-cell2cell
  role: Runs the non-negative tensor component analysis (built on tensorly) that decomposes
    the 4D-communication tensor into interpretable factors
  repo: https://github.com/earmingol/cell2cell
- name: cell2cell
  role: Provides the tensor module used to assemble the simulated communication scores
    into the 4D communication tensor and save/pickle factor results
  repo: https://github.com/earmingol/cell2cell
- name: tensorly
  role: Underlying tensor decomposition engine used by Tensor-cell2cell for the rank-scanned
    non-negative TCA
- name: Ligand-Receptor Pairs
  role: Source of the curated LR-pair set assigned to the four temporal patterns in
    the simulated tensor
  repo: https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:8409a1f4dbfdf78b2b8768dee55922d720a05dd43664070b181be06133eff215
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - Ligand-Receptor Pairs
  - Tensor-cell2cell
  - cell2cell
  - tensorly
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Factor-loading pattern recovery

## Summary

Recovers latent communication patterns embedded in a simulated 4D communication tensor (cell types x LR pairs x contexts) by running non-negative tensor component analysis with Tensor-cell2cell, selecting the rank that minimizes reconstruction error, and quantifying how factor loadings correspond to the assigned temporal patterns (oscillation, pulse, exponential decay, linear decrease).

## When to use

Use when a simulated or real 4D-communication tensor has known embedded latent patterns and you need to verify that tensor decomposition recovers them from context, LR-pair, and cell-type factor loadings.

## When NOT to use

- The tensor decomposition algorithm is not optimized for missing values, so tensors with missing cell types or LR pairs in some contexts are the wrong input for this skill
- Only cell types shared across all contexts can be included; if rare cell types absent from some samples are the analysis target, this skill will exclude them

## Inputs

- Simulated communication scores for a tensor with dimensions 3 cell types x 300 LR pairs x 12 time-point contexts
- LR-pair list drawn from the curated LewisLabUCSD/Ligand-Receptor-Pairs repository (e.g., Human-2020-Jin-LR-pairs.csv)
- Per-LR-pair temporal pattern assignments (oscillation, pulse, exponential decay, linear decrease)
- Gaussian noise levels for robustness testing

## Outputs

- Factor loadings for the context, LR-pair, and cell-type axes
- Rank-selection reconstruction errors (normalized reconstruction error vs. number of factors)
- Noise-robustness metrics of the rank-4 solution and pattern recovery
- simulation_tca_results.pkl containing loadings, rank-selection errors, and robustness metrics

## How to apply

Simulate communication scores for a tensor with dimensions 3 cell types x 300 LR pairs x 12 time-point contexts, drawing LR pairs from the curated LewisLabUCSD/Ligand-Receptor-Pairs repository and assigning each pair one of four temporal patterns (oscillation, pulse, exponential decay, linear decrease). Assemble the scores into a 4D communication tensor with cell2cell's tensor module, then run non-negative tensor component analysis via Tensor-cell2cell (built on tensorly), scanning component ranks and selecting the number of factors that minimizes the normalized reconstruction error (expected rank = 4). Extract the factor loadings for the context, LR-pair, and cell-type axes and quantify their correspondence to the four assigned patterns. Finally, repeat the decomposition with Gaussian noise added to the simulated scores to evaluate robustness of the rank-4 solution and pattern recovery, saving loadings, rank-selection errors, and noise-robustness metrics to simulation_tca_results.pkl.

## Related tools

- **Tensor-cell2cell** (Runs the non-negative tensor component analysis (built on tensorly) that decomposes the 4D-communication tensor into interpretable factors) — https://github.com/earmingol/cell2cell
- **cell2cell** (Provides the tensor module used to assemble the simulated communication scores into the 4D communication tensor and save/pickle factor results) — https://github.com/earmingol/cell2cell
- **tensorly** (Underlying tensor decomposition engine used by Tensor-cell2cell for the rank-scanned non-negative TCA)
- **Ligand-Receptor Pairs** (Source of the curated LR-pair set assigned to the four temporal patterns in the simulated tensor) — https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs

## Examples

```
import cell2cell; tensor = cell2cell.tensor.get_tensor_from_pipeline(...); t2c.run_tensor_factorization(tensor, rank=4, upper_rank=12, init='svd', factorization='nonnegative')
```

## Evaluation signals

- Rank selection on the noise-free tensor identifies 4 factors as the decomposition that best minimizes the normalized reconstruction error, matching the four embedded patterns
- Extracted context-axis loadings for each factor correlate with the assigned temporal patterns (oscillation, pulse, exponential decay, linear decrease)
- LR-pair-axis loadings for each factor concentrate on the LR pairs assigned to that factor's pattern
- The rank-4 solution and pattern correspondence persist when Gaussian noise is added to the simulated scores
- Output simulation_tca_results.pkl contains factor loadings, rank-selection reconstruction errors, and noise-robustness metrics

## Limitations

- The implemented tensor decomposition algorithm is not optimized for missing values in the communication tensor
- Tensors can only include cell types shared across all samples/contexts, potentially missing rare cell types
- Benchmarking pattern recovery is challenging due to the lack of a ground truth for real datasets; simulation is used as a surrogate evaluation

## Evidence

- [intro] We first show that Tensor-cell2cell successfully extracts temporal patterns from a simulated dataset.: "We first show that Tensor-cell2cell successfully extracts temporal patterns from a simulated dataset."
- [intro] TCA-based strategy deconvolves intercellular communication across multiple contexts: "Here, we introduce Tensor-cell2cell, a TCA-based strategy that deconvolves intercellular communication across multiple contexts and uncovers modules, or latent context-dependent patterns, of CCC."
- [methods] deconvolving complex patterns of cell-cell communication across multiple samples or contexts simultaneously into interpretable factors: "the latter is for deconvolving complex patterns of cell-cell communication across multiple samples or contexts simultaneously into interpretable factors"
- [discussion] the implemented tensor decomposition algorithm can be further optimized for missing values: "the implemented tensor decomposition algorithm can be further optimized for missing values. Since the implemented algorithm is not optimized for this purpose"
- [discussion] Choose any tool of interest, run it on each context separately, and use resulting scores to build and deconvolve a 4D-communication tensor: "One can choose any tool of interest, run it on each context separately, and use the resulting communication scores to build and deconvolve a 4D-communication tensor."
