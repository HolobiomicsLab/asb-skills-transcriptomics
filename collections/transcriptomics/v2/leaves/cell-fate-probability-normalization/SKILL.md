---
name: cell-fate-probability-normalization
description: Use when RNA velocity (scvelo spliced/unspliced AnnData) plus Cytopath
  consensus trajectory steps are available and you need per-cell alignment scores,
  trajectory assignments, pseudotime, fate probabilities, or differentiation potential
  per cell.
when_to_use_negative:
- When root and terminal cell states are not clearly identifiable — the authors do
  not recommend performing trajectory inference (and hence fate scoring) with Cytopath
  in that setting
- When the underlying RNA velocity estimation is known to be faulty or based on genes
  violating scvelo assumptions (e.g. time-dependent degradation rates), as spurious
  endpoints propagate into fate scores
- Input lacks spliced/unspliced layers or velocity estimates, so static/velocity alignment
  scores cannot be computed
tools:
- name: Cytopath
  role: Computes the alignment-score-based assignment of cells to consensus trajectory
    steps and derives normalized fate probabilities, pseudotime, and differentiation
    potential
  repo: https://github.com/aron0093/cytopath
- name: scvelo
  role: Provides the RNA velocity estimates on spliced/unspliced counts that feed
    the velocity term of the alignment score
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:da585147afa195f2acf5271244f99a804ceb2c252d028f90a373c0410e88d179
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aron0093/cytopath
  tools:
  - Cytopath
  - scvelo
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cell-fate-probability-normalization

## Summary

Scores each cell against consensus trajectory steps from Cytopath's simulated, dynamic-time-warping-aligned trajectory ensemble using a velocity-weighted alignment score, then normalizes per-cell scores into fate probabilities that sum to 1 across trajectories. The normalized fate vector additionally yields per-trajectory pseudotime and Shannon-entropy-based differentiation potential.

## When to use

Use when RNA velocity (scvelo spliced/unspliced AnnData) plus Cytopath consensus trajectory steps are available and you need per-cell alignment scores, trajectory assignments, pseudotime, fate probabilities, or differentiation potential per cell.

## When NOT to use

- When root and terminal cell states are not clearly identifiable — the authors do not recommend performing trajectory inference (and hence fate scoring) with Cytopath in that setting
- When the underlying RNA velocity estimation is known to be faulty or based on genes violating scvelo assumptions (e.g. time-dependent degradation rates), as spurious endpoints propagate into fate scores
- Input lacks spliced/unspliced layers or velocity estimates, so static/velocity alignment scores cannot be computed

## Inputs

- Processed AnnData (.h5ad) with spliced/unspliced layers and scvelo-estimated RNA velocity
- Ensemble of simulated cell state sequences (Markov-chain simulations aligned by dynamic time warping) with consensus trajectory steps from the upstream Cytopath simulation module
- Optional: pre-computed Louvain clustering used to determine terminal regions (otherwise computed internally by Cytopath)

## Outputs

- cytopath_fate_scores.csv with one row per cell containing per-cell alignment scores, trajectory step assignments, per-trajectory pseudotime, normalized fate probabilities (summing to 1 per cell), and Shannon entropy differentiation-potential scores

## How to apply

Load the processed AnnData with spliced/unspliced layers and scvelo RNA velocity together with the ensemble of Markov-chain-simulated cell state sequences aligned by dynamic time warping and their consensus steps. For every cell-trajectory-step pair, compute an alignment score combining cosine similarity of the cell's static expression profile and its velocity profile with the consensus step's profiles, weighting the velocity term per Cytopath's scoring scheme. Assign each cell to its maximum-scoring step and derive per-trajectory pseudotime as the alignment-score-weighted mean step index over cells assigned to that trajectory. Normalize each cell's alignment scores across all inferred trajectories so fate probabilities sum to 1 per cell, then compute the Shannon entropy of each cell's fate vector as differentiation potential (low entropy = committed fate, high entropy = uncommitted). Write per-cell alignment scores, assignments, pseudotime, fate probabilities, and entropy to cytopath_fate_scores.csv.

## Related tools

- **Cytopath** (Computes the alignment-score-based assignment of cells to consensus trajectory steps and derives normalized fate probabilities, pseudotime, and differentiation potential) — https://github.com/aron0093/cytopath
- **scvelo** (Provides the RNA velocity estimates on spliced/unspliced counts that feed the velocity term of the alignment score)

## Examples

```
import cytopath; cytopath.fate(prob, adata, copy=True)  # then export per-cell scores/pseudotime/fate probabilities to cytopath_fate_scores.csv
```

## Evaluation signals

- cytopath_fate_scores.csv exists and contains exactly one row per cell in the input AnnData
- Fate probabilities are non-negative and sum to 1 within each cell (row-wise normalization invariant)
- Shannon entropy scores lie in [0, log2(n_trajectories)] for every cell
- Each cell is assigned to exactly one maximum-scoring trajectory step, and pseudotime per trajectory is consistent with the score-weighted mean step index
- Fate probabilities recover expected branch associations (e.g. four terminal states in the pancreatic endocrinogenesis dataset)

## Limitations

- Cytopath pseudotime/fate scores are robust only within root and terminal probability thresholds of [0.85, 1]; outside this range behavior was reported only as 'data not shown'
- Trajectory inference and resulting fate scores are not meaningful when the underlying RNA velocity estimation is faulty, e.g. spurious root/terminal states seen in the erythroid gastrulation dataset and the scvelo simulation with time-dependent degradation rates
- Correctness of the RNA velocity estimates feeding the scores cannot be verified in every scenario, even when marker-based information is available

## Evidence

- [results] Alignment score evaluates static and velocity similarity to trajectory steps: "Assignment is based on an alignment score that evaluates for each cell the similarity of its static as well as the velocity profile with each trajectory step."
- [abstract] Cytopath Markov-chain simulation basis for the consensus steps: "Cytopath performs this task by defining a Markov chain model, simulating an ensemble of possible differentiation trajectories, and constructing a consensus trajectory."
- [readme] README on velocity-driven Markov chain model underlying trajectory steps: "Transcriptional activity information from RNA velocity of single cells is used to define a Markov chain model; simulation of this model yields an ensemble of possible differentiation trajectories"
