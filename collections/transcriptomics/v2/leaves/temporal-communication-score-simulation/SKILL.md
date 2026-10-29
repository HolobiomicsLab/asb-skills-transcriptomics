---
name: temporal-communication-score-simulation
description: 'Use when validating a context-aware CCC deconvolution method: you need
  synthetic communication scores with embedded known temporal patterns and a ground-truth
  rank to test rank selection and factor recovery.'
when_to_use_negative:
- You have real single-cell or bulk communication scores and no need for a synthetic
  ground truth — run the decomposition directly instead of simulating
- Your tensor dimensions or ground-truth pattern count differ (e.g., different number
  of embedded patterns than four), making the expected rank-4 evaluation invalid
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Tensor-cell2cell
  role: Performs non-negative tensor component analysis on the simulated 4D-communication
    tensor and enables rank scanning by normalized reconstruction error
  repo: https://github.com/earmingol/cell2cell
- name: cell2cell
  role: Provides the tensor module used to assemble communication scores into the
    4D tensor and downstream analysis pipelines
  repo: https://github.com/earmingol/cell2cell
- name: tensorly
  role: Backend tensor decomposition library on which Tensor-cell2cell's TCA is built
- name: LewisLabUCSD/Ligand-Receptor-Pairs
  role: Source of the curated LR-pair set sampled for the simulated system
  repo: https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:23ee6367c6a063ae5b3c764d952b82f5f481355c57373f7f059acbc57c030592
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - LewisLabUCSD/Ligand-Receptor-Pairs
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# temporal-communication-score-simulation

## Summary

Generate a simulated 4D cell-cell communication tensor (cell types x LR pairs x contexts/contexts x contexts) with known latent temporal patterns (e.g., oscillation, pulse, exponential decay, linear decrease) so that Tensor-cell2cell's ability to recover the correct number of factors and their loadings can be benchmarked against ground truth.

## When to use

Use when validating a context-aware CCC deconvolution method: you need synthetic communication scores with embedded known temporal patterns and a ground-truth rank to test rank selection and factor recovery.

## When NOT to use

- You have real single-cell or bulk communication scores and no need for a synthetic ground truth — run the decomposition directly instead of simulating
- Your tensor dimensions or ground-truth pattern count differ (e.g., different number of embedded patterns than four), making the expected rank-4 evaluation invalid

## Inputs

- Curated ligand-receptor pair list (e.g., Human-2020-Jin-LR-pairs.csv from LewisLabUCSD/Ligand-Receptor-Pairs)
- Simulation specification: number of cell types (3), LR pairs (300), contexts (12 time points), and the four assigned temporal pattern shapes
- Optional Gaussian noise magnitude for robustness replicates

## Outputs

- Simulated 4D-communication tensor (dimensions: cell types x LR pairs x contexts x contexts)
- Rank-selection curve of normalized reconstruction errors across component ranks
- Factor loadings for context, LR-pair, and cell-type axes with correspondence metrics to assigned patterns
- Noise-robustness metrics serialized to a pickle (e.g., simulation_tca_results.pkl)

## How to apply

Draw the ligand-receptor pair set from the curated LewisLabUCSD/Ligand-Receptor-Pairs repository and assign each LR pair one of four temporal patterns (oscillation, pulse, exponential decay, linear decrease) across 12 time-point contexts for 3 simulated cell types and 300 LR pairs. Assemble the simulated communication scores into a 4D-communication tensor using cell2cell's tensor module. Run non-negative tensor component analysis with Tensor-cell2cell (built on tensorly), scanning component ranks and selecting the rank that minimizes normalized reconstruction error (expected rank = 4). Extract context, LR-pair, and cell-type factor loadings and quantify their correspondence to the assigned patterns; repeat with Gaussian noise added to evaluate robustness of the rank-4 solution. Save loadings, rank-selection errors, and noise-robustness metrics (e.g., simulation_tca_results.pkl).

## Related tools

- **Tensor-cell2cell** (Performs non-negative tensor component analysis on the simulated 4D-communication tensor and enables rank scanning by normalized reconstruction error) — https://github.com/earmingol/cell2cell
- **cell2cell** (Provides the tensor module used to assemble communication scores into the 4D tensor and downstream analysis pipelines) — https://github.com/earmingol/cell2cell
- **tensorly** (Backend tensor decomposition library on which Tensor-cell2cell's TCA is built)
- **LewisLabUCSD/Ligand-Receptor-Pairs** (Source of the curated LR-pair set sampled for the simulated system) — https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs

## Examples

```
import cell2cell; tensor = cell2cell.tensor.get_tensor_from_communication_scores(scores); t2c.run_tensor_factorization(tensor, rank=4, backend='tensorly')
```

## Evaluation signals

- Rank selection: the decomposition minimizing normalized reconstruction error should occur at rank 4, matching the four introduced communication patterns
- Correspondence between extracted LR-pair/context factor loadings and the assigned oscillation, pulse, exponential-decay, and linear-decrease temporal shapes
- Rank-4 solution is stable when Gaussian noise is added to the simulated scores (noise-robustness metrics quantified)
- Simulated tensor has the specified dimensions (3 cell types x 300 LR pairs x 12 time-point contexts) before assembly into 4D form
- All loadings, rank-selection errors, and robustness metrics are present in the saved pickle (simulation_tca_results.pkl)

## Limitations

- The simulated system (3 cell types, 300 LR pairs, 12 contexts, 4 patterns) is a simplification; the article notes benchmarking CCC prediction tools is challenging due to the lack of a ground truth in real data
- The implemented tensor decomposition algorithm is not optimized for missing values, so simulations must produce a complete tensor (no missing cell types or LR pairs across contexts)

## Evidence

- [other] Simulate communication scores for a tensor with dimensions 3 cell types x 300 LR pairs x 12 time-point contexts: [evidence span withheld — anchor status: not_found]
- [intro] We first show that Tensor-cell2cell successfully extracts temporal patterns from a simulated dataset: "We first show that Tensor-cell2cell successfully extracts temporal patterns from a simulated dataset."
- [other] In the simulated 4D-communication tensor, four factors gave the decomposition that best minimized error: [evidence span withheld — anchor status: not_found]
- [discussion] the implemented tensor decomposition algorithm can be further optimized for missing values: "the implemented tensor decomposition algorithm can be further optimized for missing values. Since the implemented algorithm is not optimized for this purpose"
- [readme] Find a curated list of ligand-receptor pairs for your analyses at our GitHub Repository: "This compendium has been created to collect lists of ligand-receptor pairs
and interactions for surface-secreted proteins that are available in literature."
