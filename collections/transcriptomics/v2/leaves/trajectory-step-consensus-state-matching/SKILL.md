---
name: trajectory-step-consensus-state-matching
description: Use when an AnnData object with spliced/unspliced layers and scvelo-estimated
  RNA velocity has already been processed through Cytopath's Markov simulation and
  consensus-step construction, and you need cell-level scores against the inferred
  trajectories.
when_to_use_negative:
- The underlying RNA velocity estimation is faulty or unverifiable (e.g. time-dependent
  degradation rates), since Cytopath inference is not meaningful in that case
- Root and terminal cell states are not clearly identifiable (e.g. fuzzy root probabilities
  with no discernible pattern, as in the hepatocyte zonation dataset)
- The input lacks spliced/unspliced layers, so no RNA velocity profile can be computed
  for the velocity-weighted alignment score
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cytopath
  role: Computes the cell-to-trajectory-step alignment scores, maximum-score assignments,
    pseudotime, fate probabilities, and entropy scores
  repo: https://github.com/aron0093/cytopath
- name: scvelo
  role: Supplies the RNA velocity estimates (and latent time baseline) consumed by
    the alignment scoring layer
- name: cytopath-notebooks
  role: Demonstration notebooks showing Cytopath applied to public scRNA-seq datasets
  repo: https://github.com/aron0093/cytopath-notebooks
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:9962f75924458ec3d358314622305b7cdf19c5ac253c15f4d3f6e329afbf515c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aron0093/cytopath
  tools:
  - Cytopath
  - cytopath-notebooks
  - scvelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# trajectory-step-consensus-state-matching

## Summary

Assigns individual cells to the consensus trajectory steps of Cytopath by scoring each cell's static expression and RNA velocity profile against every consensus step, yielding per-cell trajectory assignments, pseudotime, fate probabilities, and differentiation-potential (entropy) scores.

## When to use

Use when an AnnData object with spliced/unspliced layers and scvelo-estimated RNA velocity has already been processed through Cytopath's Markov simulation and consensus-step construction, and you need cell-level scores against the inferred trajectories.

## When NOT to use

- The underlying RNA velocity estimation is faulty or unverifiable (e.g. time-dependent degradation rates), since Cytopath inference is not meaningful in that case
- Root and terminal cell states are not clearly identifiable (e.g. fuzzy root probabilities with no discernible pattern, as in the hepatocyte zonation dataset)
- The input lacks spliced/unspliced layers, so no RNA velocity profile can be computed for the velocity-weighted alignment score

## Inputs

- Processed AnnData object (h5ad/loom) with spliced/unspliced layers and scvelo-estimated RNA velocity
- Ensemble of Cytopath simulated cell state sequences aligned by dynamic time warping
- Consensus trajectory steps (per-step averaged expression/velocity states)
- Optional pre-computed cell clustering (otherwise Louvain clustering is computed internally)

## Outputs

- Per-cell alignment scores for every cell-trajectory-step pair
- Per-cell maximum-scoring trajectory-step assignment
- Per-trajectory pseudotime values per cell
- Per-cell fate probabilities (normalized to sum to 1 across trajectories)
- Per-cell differentiation-potential Shannon entropy scores
- cytopath_fate_scores.csv (one row per cell)

## How to apply

Load the processed AnnData (h5ad/loom with spliced and unspliced layers) together with the ensemble of simulated cell state sequences aligned by dynamic time warping and their averaged consensus trajectory steps. For every cell-trajectory-step pair, compute an alignment score that combines cosine similarity of the cell's static expression profile with its velocity profile against the consensus step's profiles, with the velocity term weighted in. Assign each cell to its maximum-scoring step, then compute per-trajectory pseudotime as the alignment-score-weighted mean step index over assigned cells. Normalize each cell's scores across trajectories so fate probabilities sum to 1, and compute the Shannon entropy of the fate-probability vector as differentiation potential (low entropy = committed fate, high entropy = uncommitted). Rationale: static profiles alone are ambiguous with respect to directionality, so velocity similarity is required to place cells along directed trajectories.

## Related tools

- **Cytopath** (Computes the cell-to-trajectory-step alignment scores, maximum-score assignments, pseudotime, fate probabilities, and entropy scores) — https://github.com/aron0093/cytopath
- **scvelo** (Supplies the RNA velocity estimates (and latent time baseline) consumed by the alignment scoring layer)
- **cytopath-notebooks** (Demonstration notebooks showing Cytopath applied to public scRNA-seq datasets) — https://github.com/aron0093/cytopath-notebooks

## Examples

```
pip install cytopath
```

## Evaluation signals

- cytopath_fate_scores.csv exists with exactly one row per cell
- Fate probabilities are non-negative and sum to 1 per cell across all inferred trajectories
- Entropy scores lie in [0, log2(n_trajectories)]
- Every cell has a finite alignment score for at least one trajectory step, and each cell's assigned step is its argmax
- Per-trajectory pseudotime correlates with known cell-type ordering (e.g. high Spearman correlation and robustness across multiple runs, as on the hippocampus dataset)

## Limitations

- Trajectory inference and downstream cell scoring are not meaningful when the RNA velocity estimation itself is faulty; simplifying assumptions (time-invariant transcription/splicing/degradation rates, one regulatory regime per gene) can produce spurious root/terminal states (e.g. erythroid gastrulation dataset)
- Not recommended when root and terminal cell states are not clearly identifiable
- Correctness of the velocity estimates cannot always be verified, even when independent marker-based information is available

## Evidence

- [results] Assignment of cells to trajectory steps based on an alignment score of static and velocity profiles: "Assignment is based on an alignment score that evaluates for each cell the similarity of its static as well as the velocity profile with each trajectory step."
- [results] Consensus states are per-step averages over aligned simulations: "Cell states at every step of the ensemble of aligned simulations are averaged, and the average value is considered the consensus state of the trajectory at the particular step"
- [abstract] Simulation-based construction of consensus trajectories: "Cytopath performs this task by defining a Markov chain model, simulating an ensemble of possible differentiation trajectories, and constructing a consensus trajectory."
- [readme] Cytopath uses RNA velocity to define the Markov chain model: "Transcriptional activity information from RNA velocity of single cells is used to define a Markov chain model; simulation of this model yields an ensemble of possible differentiation trajectories"
- [discussion] Do not apply when root/terminal states are unclear: "We do not recommend performing trajectory inference with Cytopath when the root and terminal cell states are not clearly identifiable."
