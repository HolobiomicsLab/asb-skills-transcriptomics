---
name: noise-perturbation-robustness-evaluation
description: Use when a simulated tensor with known embedded patterns (e.g., 4 temporal
  patterns across 3 cell types, 300 LR pairs, 12 contexts) must be shown recoverable,
  or when rank selection must be stress-tested against measurement noise.
when_to_use_negative:
- 'The tensor contains missing values: the article notes ''the implemented tensor
  decomposition algorithm is not optimized'' for missing values, so noise-robustness
  conclusions would not transfer'
- 'There is no ground truth (e.g., real biological tensors): the article stresses
  that ''benchmarking CCC prediction tools is challenging due to the lack of a ground
  truth'', so pattern-recovery accuracy cannot be judged'
- The communication matrix is a single context (2D), where context-aware TCA and its
  rank-robustness analysis do not apply
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Tensor-cell2cell
  role: Non-negative tensor component analysis backend for scanning component ranks
    and extracting factor loadings from the 4D communication tensor
  repo: https://github.com/earmingol/cell2cell
- name: cell2cell
  role: Tensor module used to assemble the simulated communication scores into the
    4D tensor and run downstream factor analyses
  repo: https://github.com/earmingol/cell2cell
- name: tensorly
  role: Tensor-decomposition engine underlying Tensor-cell2cell used for the rank
    scan and reconstruction-error computation
- name: LewisLabUCSD/Ligand-Receptor-Pairs
  role: Source of the curated LR-pair list used to populate the simulated tensor's
    LR-pair axis
  repo: https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:4a4ca11a8a19d265d938a5507743fceaecc7e3eff09fa111378178ea9ee81f8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - LewisLabUCSD/Ligand-Receptor-Pairs
  - Tensor-cell2cell
  - cell2cell
  - tensorly
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# noise-perturbation-robustness-evaluation

## Summary

Evaluate whether a decomposition (e.g., Tensor-cell2cell non-negative TCA on a 4D communication tensor) recovers the true latent structure by adding Gaussian noise to simulated scores and re-running the rank scan. It grounds confidence in the selected rank and factor loadings before applying the method to real biological contexts.

## When to use

Use when a simulated tensor with known embedded patterns (e.g., 4 temporal patterns across 3 cell types, 300 LR pairs, 12 contexts) must be shown recoverable, or when rank selection must be stress-tested against measurement noise.

## When NOT to use

- The tensor contains missing values: the article notes 'the implemented tensor decomposition algorithm is not optimized' for missing values, so noise-robustness conclusions would not transfer
- There is no ground truth (e.g., real biological tensors): the article stresses that 'benchmarking CCC prediction tools is challenging due to the lack of a ground truth', so pattern-recovery accuracy cannot be judged
- The communication matrix is a single context (2D), where context-aware TCA and its rank-robustness analysis do not apply

## Inputs

- Simulated 4D communication tensor (cell types x LR pairs x contexts, with embedded ground-truth patterns)
- LR-pair list drawn from the LewisLabUCSD/Ligand-Receptor-Pairs curated repository
- Pattern-assignment map (each LR pair assigned to oscillation, pulse, exponential-decay, or linear-decrease)

## Outputs

- Rank-scan table of normalized reconstruction errors showing the optimal rank (expected 4)
- Factor loadings for the context, LR-pair and cell-type axes
- Noise-robustness metrics comparing clean vs. Gaussian-noise-perturbed decompositions
- Pickled results file (e.g., simulation_tca_results.pkl)

## How to apply

First build a ground-truth simulation: draw LR pairs from the curated LewisLabUCSD/Ligand-Receptor-Pairs compendium, assign each LR pair one of four temporal patterns (oscillation, pulse, exponential decay, linear decrease), and assemble a 4D communication tensor (3 cell types x 300 LR pairs x 12 time-point contexts) with cell2cell's tensor module. Run non-negative tensor component analysis with Tensor-cell2cell (built on tensorly), scanning component ranks and selecting the rank that minimizes the normalized reconstruction error (expected rank = 4). Then repeat the full decomposition pipeline with Gaussian noise added to the simulated scores and verify that the rank-4 solution and the context/LR-pair/cell-type loadings remain consistent with the four assigned patterns. Persist factor loadings, rank-selection reconstruction errors, and noise-robustness metrics (e.g., to simulation_tca_results.pkl) so recovery can be quantified against the known ground truth.

## Related tools

- **Tensor-cell2cell** (Non-negative tensor component analysis backend for scanning component ranks and extracting factor loadings from the 4D communication tensor) — https://github.com/earmingol/cell2cell
- **cell2cell** (Tensor module used to assemble the simulated communication scores into the 4D tensor and run downstream factor analyses) — https://github.com/earmingol/cell2cell
- **tensorly** (Tensor-decomposition engine underlying Tensor-cell2cell used for the rank scan and reconstruction-error computation)
- **LewisLabUCSD/Ligand-Receptor-Pairs** (Source of the curated LR-pair list used to populate the simulated tensor's LR-pair axis) — https://github.com/LewisLabUCSD/Ligand-Receptor-Pairs

## Examples

```
pip install cell2cell  # then scan ranks via Tensor-cell2cell TCA and re-run with Gaussian noise to log rank-selection errors
```

## Evaluation signals

- The rank scan of normalized reconstruction error reaches its minimum at rank 4, matching the four embedded communication patterns
- Extracted context-axis loadings reproduce the assigned temporal shapes (oscillation, pulse, exponential decay, linear decrease)
- Adding Gaussian noise to the simulated scores does not change the selected rank; robustness metrics quantify stability of the rank-4 solution
- LR-pair and cell-type loadings assign each simulated LR pair predominantly to its correct pattern factor
- All loadings, rank-selection errors, and noise-robustness metrics are persisted (simulation_tca_results.pkl) and reproducible across runs

## Limitations

- Recovery accuracy depends on the simulation's ground truth; benchmarking CCC tools is challenging due to lack of ground truth for real data
- The implemented tensor decomposition algorithm is not optimized for missing values, so tensors with incompletely shared cell types or LR pairs cannot be robustly evaluated
- Rank selection relies on minimizing normalized reconstruction error and can be sensitive to the magnitude of added Gaussian noise

## Evidence

- [intro] Simulated temporal-pattern validation: "We first show that Tensor-cell2cell successfully extracts temporal patterns from a simulated dataset."
- [intro] Method performance rationale: "we show that Tensor-cell2cell is faster, demands less memory and can achieve better accuracy in separating context-specific information"
- [abstract] Robustness claim grounding: "As such, Tensor-cell2cell robustly improves upon and extends the analytical capabilities of existing tools"
- [readme] Tool installation for reproduction: [evidence span withheld — anchor status: not_found]
- [discussion] Benchmarking limitation: "we note that benchmarking CCC prediction tools is challenging due to the lack of a ground truth"
