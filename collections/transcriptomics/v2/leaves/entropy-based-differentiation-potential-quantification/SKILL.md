---
name: entropy-based-differentiation-potential-quantification
description: Use when you have per-cell alignment scores from Cytopath trajectory
  inference and need per-cell fate probabilities, differentiation potential, or identification
  of uncommitted vs. committed cell states.
when_to_use_negative:
- When the underlying RNA velocity estimation is faulty — the authors do not expect
  trajectory inference (and hence fate scoring) to be meaningful in that case
- When root and terminal cell states are not clearly identifiable (e.g., fuzzy root
  cell probabilities with no discernible pattern), which the authors explicitly do
  not recommend for Cytopath
- When no per-cell alignment scores over multiple trajectories exist — entropy over
  a single-trajectory assignment is degenerate
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cytopath
  role: Produces the cell-to-trajectory-step alignment scores (static + velocity similarity)
    that are normalized into fate probabilities and summarized as entropy-based differentiation
    potential
  repo: https://github.com/aron0093/cytopath
- name: scvelo
  role: Supplies the RNA velocity estimates (from spliced/unspliced layers) that make
    the alignment scores directional and fate-aware
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:fa60228040a6eda42bc2289a9dc422f508c23759a8865514799b8a107d7b2377
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aron0093/cytopath
  tools:
  - Cytopath
  - scvelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# entropy-based-differentiation-potential-quantification

## Summary

Quantifies each cell's differentiation potential by normalizing Cytopath cell-to-trajectory alignment scores into fate probabilities and computing the Shannon entropy of the fate-probability vector, where low entropy signals a committed fate and high entropy an uncommitted, multi-potent state.

## When to use

Use when you have per-cell alignment scores from Cytopath trajectory inference and need per-cell fate probabilities, differentiation potential, or identification of uncommitted vs. committed cell states.

## When NOT to use

- When the underlying RNA velocity estimation is faulty — the authors do not expect trajectory inference (and hence fate scoring) to be meaningful in that case
- When root and terminal cell states are not clearly identifiable (e.g., fuzzy root cell probabilities with no discernible pattern), which the authors explicitly do not recommend for Cytopath
- When no per-cell alignment scores over multiple trajectories exist — entropy over a single-trajectory assignment is degenerate

## Inputs

- AnnData (.h5ad) object with spliced/unspliced layers and scvelo-estimated RNA velocity
- Ensemble of simulated cell state sequences (Markov-chain simulations aligned by dynamic time warping) with consensus trajectory steps from Cytopath
- Per-cell alignment scores against each trajectory step

## Outputs

- cytopath_fate_scores.csv with one row per cell: alignment scores, trajectory assignments, pseudotime, fate probabilities, entropy scores
- Per-cell fate probability vector (sums to 1 across trajectories)
- Per-cell differentiation potential (Shannon entropy, in [0, log2(n_trajectories)])

## How to apply

After Cytopath assigns each cell to trajectory steps using an alignment score that combines static expression and RNA velocity profile similarity (cosine-similarity-based, velocity-weighted), normalize each cell's alignment scores across all inferred trajectories so fate probabilities sum to 1 per cell. Compute the Shannon entropy of each cell's fate-probability vector as its differentiation potential, interpreting low entropy as a committed fate and high entropy as uncommitted. Write per-cell alignment scores, trajectory assignments, pseudotime (alignment-score-weighted mean step index), fate probabilities, and entropy scores to cytopath_fate_scores.csv. This step depends on the upstream Cytopath Markov-chain simulations, dynamic time warping alignment, and consensus trajectory steps, with spliced/unspliced layers and scvelo-estimated RNA velocity supplied in an AnnData object.

## Related tools

- **Cytopath** (Produces the cell-to-trajectory-step alignment scores (static + velocity similarity) that are normalized into fate probabilities and summarized as entropy-based differentiation potential) — https://github.com/aron0093/cytopath
- **scvelo** (Supplies the RNA velocity estimates (from spliced/unspliced layers) that make the alignment scores directional and fate-aware)

## Examples

```
import pandas as pd, numpy as np; from scipy.stats import entropy; df = pd.read_csv('cytopath_fate_scores.csv'); df['diff_potential'] = entropy(df[[c for c in df if c.startswith('fate_')]].values, base=2, axis=1)
```

## Evaluation signals

- cytopath_fate_scores.csv exists with exactly one row per cell
- Fate probabilities are non-negative and sum to 1 per cell
- Entropy scores lie within [0, log2(n_trajectories)]
- Each cell's assigned trajectory step corresponds to its maximum alignment score
- Committed populations (e.g., terminal lineage cells in the pancreatic endocrinogenesis dataset's alpha/beta/gamma/delta states) show lower entropy than progenitor/cycling cells

## Limitations

- Faulty RNA velocity estimation (e.g., time-dependent degradation rates in a scvelo-simulated dataset) leads to spurious root/terminal states and invalid downstream fate scores
- For the erythroid gastrulation dataset, spurious root points were inferred toward the terminus of the expected process, corrupting pseudotime and fate assignment
- Pseudotime robustness to root/terminal probability thresholds ([0.85, 1]) was reported only as 'data not shown'

## Evidence

- [results] Alignment score definition: "Assignment is based on an alignment score that evaluates for each cell the similarity of its static as well as the velocity profile with each trajectory step."
- [abstract] Cytopath overview: "Cytopath performs this task by defining a Markov chain model, simulating an ensemble of possible differentiation trajectories, and constructing a consensus trajectory."
- [readme] README: velocity-based Markov chain model: "Transcriptional activity information from RNA velocity of single cells is used to define a Markov chain model; simulation of this model yields an ensemble of possible differentiation trajectories"
- [readme] README: statistical analysis of topology: "Subsequent statistical analysis reveals the topological and molecular characteristics of the differentiation process."
- [discussion] Discussion: faulty velocity caveat: "We do not expect that trajectory inference with Cytopath is meaningful when the underlying RNA velocity estimation itself is faulty."
