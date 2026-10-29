---
name: rna-velocity-alignment-score-computation
description: Use when you have an AnnData object with spliced/unspliced layers, scvelo
  RNA velocity, and an ensemble of DTW-aligned simulated trajectories, and need per-cell
  scores, pseudotime, and cell fate probabilities.
when_to_use_negative:
- Root and terminal cell states are not clearly identifiable from the RNA velocity
  field — the authors explicitly do not recommend Cytopath trajectory inference in
  this case
- The underlying RNA velocity estimation is faulty (e.g. spurious root/terminal states
  as in the erythroid gastrulation dataset or with time-dependent degradation rates),
  since downstream alignment scores inherit the error
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cytopath
  role: 'Implements the alignment-score layer: scores static and velocity similarity
    of each cell against consensus trajectory steps and derives pseudotime and fate
    scores'
  repo: https://github.com/aron0093/cytopath
- name: scvelo
  role: Provides the RNA velocity estimates (and latent time baseline) supplied as
    the velocity profiles used in the alignment scoring
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:0eb051199066c4426b3209930e49ba98c92c8b7d0b2e6928ee406448decdef1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aron0093/cytopath
  tools:
  - Cytopath
  - scvelo
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# rna-velocity-alignment-score-computation

## Summary

Compute Cytopath's cell-to-trajectory alignment scores that evaluate each cell's static expression and RNA velocity similarity against consensus trajectory steps, then use the maximum-scoring assignment to derive per-trajectory pseudotime, fate probabilities, and differentiation potential.

## When to use

Use when you have an AnnData object with spliced/unspliced layers, scvelo RNA velocity, and an ensemble of DTW-aligned simulated trajectories, and need per-cell scores, pseudotime, and cell fate probabilities.

## When NOT to use

- Root and terminal cell states are not clearly identifiable from the RNA velocity field — the authors explicitly do not recommend Cytopath trajectory inference in this case
- The underlying RNA velocity estimation is faulty (e.g. spurious root/terminal states as in the erythroid gastrulation dataset or with time-dependent degradation rates), since downstream alignment scores inherit the error

## Inputs

- Processed AnnData (.h5ad) with spliced/unspliced layers and scvelo-estimated RNA velocity
- Ensemble of Markov-chain-simulated cell state sequences aligned by dynamic time warping
- Consensus trajectory steps (averaged cell states per step per trajectory)
- Optional pre-computed cell clustering (otherwise Louvain clustering computed internally)

## Outputs

- cytopath_fate_scores.csv with per-cell alignment scores, trajectory assignments, pseudotime values, fate probabilities, and entropy scores
- Per-trajectory pseudotime values per cell
- Cell fate probability vectors (normalized to sum to 1 per cell)
- Shannon-entropy-based differentiation potential per cell

## How to apply

Load the processed AnnData (spliced/unspliced layers, scvelo-estimated RNA velocity) together with the ensemble of Markov-simulated cell state sequences aligned by dynamic time warping and their averaged consensus states. For every cell-trajectory-step pair, compute the alignment score as the combined cosine-similarity-based, velocity-weighted similarity of the cell's static expression and velocity profiles with the consensus step's profiles. Assign each cell to its maximum-scoring trajectory step, and compute per-trajectory pseudotime as the alignment-score-weighted mean step index over the cells assigned to that trajectory. Normalize alignment scores across all inferred trajectories so fate probabilities sum to 1 per cell, then compute the Shannon entropy of each cell's fate-probability vector as its differentiation potential (low entropy = committed fate, high entropy = uncommitted).

## Related tools

- **Cytopath** (Implements the alignment-score layer: scores static and velocity similarity of each cell against consensus trajectory steps and derives pseudotime and fate scores) — https://github.com/aron0093/cytopath
- **scvelo** (Provides the RNA velocity estimates (and latent time baseline) supplied as the velocity profiles used in the alignment scoring)

## Examples

```
import cytopath; cypath = cytopath.path_inference(adata, n_jobs=8); cytopath.cytopaudit(adata, cypath)
```

## Evaluation signals

- cytopath_fate_scores.csv exists with exactly one row per cell in the AnnData object
- Fate probabilities are non-negative and sum to 1 within each cell across trajectories
- Shannon entropy scores lie in the interval [0, log2(n_trajectories)]
- Each cell is assigned to exactly one maximum-scoring trajectory step
- Per-trajectory pseudotime shows high Spearman correlation with known cell-type ordering per lineage (as reported for the developing mouse hippocampus dataset) and is robust across multiple runs

## Limitations

- Cytopath pseudotime robustness to root and terminal probability thresholds within [0.85, 1] was reported only as 'data not shown'
- RNA velocity estimation can be unrepresentative of true transcriptional dynamics due to simplifying assumptions (time-invariant transcription/splicing/degradation rates, single regulatory regime per gene)
- Incorrect RNA velocity estimation leads to spurious inference of root and terminal states, as observed in the erythroid gastrulation dataset and in a simulated dataset with time-dependent degradation rates

## Evidence

- [results] Alignment score definition: "Assignment is based on an alignment score that evaluates for each cell the similarity of its static as well as the velocity profile with each trajectory step."
- [results] Consensus state construction upstream of scoring: "Cell states at every step of the ensemble of aligned simulations are averaged, and the average value is considered the consensus state of the trajectory at the particular step"
- [abstract] Method scope statement: "Cytopath performs this task by defining a Markov chain model, simulating an ensemble of possible differentiation trajectories, and constructing a consensus trajectory."
- [results] Pseudotime validation in hippocampus: "The Spearman correlation between pseudotime inferred by Cytopath for each trajectory to known ordering of cell types is high (Figure 2D) and robust across multiple runs (Figure 2E)."
- [discussion] Failure mode: faulty velocity: "We do not expect that trajectory inference with Cytopath is meaningful when the underlying RNA velocity estimation itself is faulty."
