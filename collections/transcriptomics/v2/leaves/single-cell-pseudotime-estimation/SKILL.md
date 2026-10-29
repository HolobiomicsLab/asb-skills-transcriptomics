---
name: single-cell-pseudotime-estimation
description: Use when a processed AnnData with spliced/unspliced layers and scvelo
  RNA velocity plus Cytopath consensus trajectory steps are available and per-cell
  pseudotime and cell fate probabilities must be quantified.
when_to_use_negative:
- The underlying RNA velocity estimation itself is faulty (e.g., spurious root/terminal
  states as seen in the erythroid gastrulation dataset); the authors do not expect
  trajectory inference to be meaningful in this case.
- Root and terminal cell states are not clearly identifiable (e.g., fuzzy root cell
  probability without discernible pattern as in the hepatocyte zonation dataset);
  the authors do not recommend performing trajectory inference with Cytopath here.
- Input lacks velocity information — static expression profiles alone are ambiguous
  with respect to directionality of cell state transitions, undermining root/terminal
  assignment and pseudotime ordering.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: Cytopath
  role: Performs Markov-chain simulation, dynamic time warping alignment, consensus
    trajectory construction, and the alignment-score-based cell assignment that yields
    pseudotime and fate scores
  repo: https://github.com/aron0093/cytopath
- name: scvelo
  role: Provides the RNA velocity estimates (and spliced/unspliced layers) that weight
    the alignment score and supply directionality to cell-to-trajectory assignment
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:51c0823961f8ef32ae39a208fbb0fcc99ee5caeecfbc7581036d3aa41b07a136
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aron0093/cytopath
  tools:
  - Cytopath
  - scvelo
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# single-cell-pseudotime-estimation

## Summary

Assigns each cell to inferred trajectory steps of Cytopath's consensus trajectories via alignment scores combining static expression and RNA velocity similarity, then derives per-cell pseudotime, fate probabilities, and differentiation potential (Shannon entropy of fate vector).

## When to use

Use when a processed AnnData with spliced/unspliced layers and scvelo RNA velocity plus Cytopath consensus trajectory steps are available and per-cell pseudotime and cell fate probabilities must be quantified.

## When NOT to use

- The underlying RNA velocity estimation itself is faulty (e.g., spurious root/terminal states as seen in the erythroid gastrulation dataset); the authors do not expect trajectory inference to be meaningful in this case.
- Root and terminal cell states are not clearly identifiable (e.g., fuzzy root cell probability without discernible pattern as in the hepatocyte zonation dataset); the authors do not recommend performing trajectory inference with Cytopath here.
- Input lacks velocity information — static expression profiles alone are ambiguous with respect to directionality of cell state transitions, undermining root/terminal assignment and pseudotime ordering.

## Inputs

- Processed AnnData object (.h5ad) with spliced/unspliced layers and scvelo-estimated RNA velocity
- Ensemble of Markov-chain simulated cell state sequences aligned by dynamic time warping
- Consensus trajectory steps (averaged cell states at every step of aligned simulations)
- Optionally a pre-computed cell clustering (otherwise Louvain clustering is computed internally for terminal regions)

## Outputs

- cytopath_fate_scores.csv with one row per cell containing per-trajectory alignment scores, maximum-scoring trajectory/step assignments, per-trajectory pseudotime, fate probabilities (summing to 1 per cell), and Shannon-entropy differentiation potential scores

## How to apply

Load the processed AnnData (spliced/unspliced layers, scvelo-estimated velocity) together with the ensemble of Markov-chain simulated cell state sequences aligned by dynamic time warping and their averaged consensus steps. For every cell-trajectory-step pair, compute an alignment score that evaluates the similarity of the cell's static expression profile and its velocity profile with the consensus step's profiles, using a cosine-similarity-based, velocity-weighted score, then assign each cell to its maximum-scoring step and compute per-trajectory pseudotime as the alignment-score-weighted mean step index. Normalize each cell's alignment scores across all inferred trajectories so fate probabilities sum to 1, and compute the Shannon entropy of each cell's fate-probability vector as differentiation potential (low entropy = committed fate, high entropy = uncommitted). Write per-cell alignment scores, trajectory assignments, pseudotime, fate probabilities, and entropy scores to cytopath_fate_scores.csv, validating one row per cell, non-negative fate probabilities summing to 1, and entropy in [0, log2(n_trajectories)].

## Related tools

- **Cytopath** (Performs Markov-chain simulation, dynamic time warping alignment, consensus trajectory construction, and the alignment-score-based cell assignment that yields pseudotime and fate scores) — https://github.com/aron0093/cytopath
- **scvelo** (Provides the RNA velocity estimates (and spliced/unspliced layers) that weight the alignment score and supply directionality to cell-to-trajectory assignment)

## Examples

```
import cytopath; cytopath.terminal_states(adata); cytopath.cytopath(adata, n_simulations=100)  # then read cytopath_fate_scores.csv
```

## Evaluation signals

- cytopath_fate_scores.csv exists and contains exactly one row per cell in the input AnnData
- Fate probabilities are non-negative and sum to 1 for every cell
- Shannon entropy scores lie within [0, log2(n_trajectories)]
- Per-trajectory pseudotime shows high Spearman correlation with known ordering of cell types per lineage (as reported for the hippocampus dataset) and is robust across multiple runs
- Each cell's assigned trajectory step is the argmax of its alignment-score vector over steps of that trajectory

## Limitations

- RNA velocity estimation can be unrepresentative of true transcriptional dynamics due to time-invariant transcription/splicing/degradation rate assumptions and a single regulatory regime per gene
- Incorrect RNA velocity estimation leads to spurious root and terminal cell states (erythroid gastrulation dataset, simulated scvelo dataset with time-dependent degradation rates)
- Pseudotime robustness to root and terminal probability thresholds within [0.85, 1] was reported only as 'data not shown'
- Correctness of RNA velocity estimation cannot be verified in every scenario, even when independent marker-based information is available

## Evidence

- [results] Assignment is based on an alignment score that evaluates for each cell the similarity of its static as well as the velocity profile with each trajectory step.: "Assignment is based on an alignment score that evaluates for each cell the similarity of its static as well as the velocity profile with each trajectory step."
- [results] Cell states at every step of the ensemble of aligned simulations are averaged, and the average value is considered the consensus state of the trajectory at the particular step: "Cell states at every step of the ensemble of aligned simulations are averaged, and the average value is considered the consensus state of the trajectory at the particular step"
- [results] The Spearman correlation between pseudotime inferred by Cytopath for each trajectory to known ordering of cell types is high (Figure 2D) and robust across multiple runs (Figure 2E).: "The Spearman correlation between pseudotime inferred by Cytopath for each trajectory to known ordering of cell types is high (Figure 2D) and robust across multiple runs (Figure 2E)."
- [discussion] We found pseudotime inferred by Cytopath to be robust to root and terminal probability thresholds within a range of [0.85, 1] (data not shown).: "We found pseudotime inferred by Cytopath to be robust to root and terminal probability thresholds within a range of [0.85, 1] (data not shown)."
- [readme] Cytopath is a method for trajectory inference with single-cell RNA sequencing data. Transcriptional activity information from RNA velocity of single cells is used to define a Markov chain model: "Cytopath is a method for trajectory inference with single-cell RNA sequencing data. Transcriptional activity information from RNA velocity of single cells is used to define a Markov chain model"
