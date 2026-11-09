---
name: terminal-state-and-root-cell-definition
description: Use when a dandelionR trajectory run on a pseudobulked VDJ feature space
  (pb.milo) needs an explicit root and terminal states before DiffusionMap/DPT and
  markovProbability can be invoked.
when_to_use_negative:
- The trajectory is cyclic (e.g. B cell receptor editing), which cannot be represented
  by absorbing Markov chains with fixed absorbing terminal states
- The data topology is not bifurcating or linear — the destiny-based diffusion map
  is only suitable for bifurcation or linear topologies, limiting this root/tip strategy
- The input is still single cells or lacks a pseudobulked VDJ feature-space PCA (e.g.
  no vdjPseudobulk output), since extreme-cell selection is defined on the pseudobulk
  embedding
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: dandelionR
  role: Implements waypoint selection, absorbing Markov chain construction, branch-probability
    calculation and projection (markovProbability) that consume the defined root and
    terminal states
  repo: github.com/tuonglab/dandelionR
- name: destiny
  role: Computes the diffusion map and runs DPT with the tips argument set to c(root,
    branch.tips) to obtain anchored diffusion pseudotime
- name: miloR
  role: Provides the pseudobulked neighbourhood object (pb.milo) whose PCA embedding
    is used to locate the extreme root and branch-tip cells
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:e8724102001ba08386768d3a8baedf9403f4ea6a62ff5f6b29153f5490808fea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Dandelion
  - R
  - dandelionR
  - destiny
  - miloR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# terminal-state-and-root-cell-definition

## Summary

Defines the root cell and terminal (branch-tip) states geometrically from the pseudobulk VDJ feature-space PCA embedding, so that destiny's DPT and the Palantir-style absorbing Markov chain can compute pseudotime and branch probabilities.

## When to use

Use when a dandelionR trajectory run on a pseudobulked VDJ feature space (pb.milo) needs an explicit root and terminal states before DiffusionMap/DPT and markovProbability can be invoked.

## When NOT to use

- The trajectory is cyclic (e.g. B cell receptor editing), which cannot be represented by absorbing Markov chains with fixed absorbing terminal states
- The data topology is not bifurcating or linear — the destiny-based diffusion map is only suitable for bifurcation or linear topologies, limiting this root/tip strategy
- The input is still single cells or lacks a pseudobulked VDJ feature-space PCA (e.g. no vdjPseudobulk output), since extreme-cell selection is defined on the pseudobulk embedding

## Inputs

- pb.milo: pseudobulked miloR object with a Feature_space assay (VDJ feature space) and a 20-component PCA embedding
- Precomputed destiny DiffusionMap on the feature space (n_pcs = 50, n_eigs = 10)
- User-chosen branch labels for the terminal states (e.g. 'CD8+T', 'CD4+T')

## Outputs

- Named vectors root (single pseudobulk cell ID) and branch.tips (one cell ID per terminal branch)
- diffusion pseudotime stored as a 'pseudotime' column in pb.milo colData
- Branch-probability columns named after the terminal states ('CD8+T', 'CD4+T') in pb.milo colData

## How to apply

Work on the 20-component PCA embedding of the Feature_space assay of the pseudobulked miloR object produced by vdjPseudobulk. Pick the terminal (branch-tip) cells as the top-most and bottom-most cells along PC2 — in the dandelionR abT tutorial these become the CD8+T and CD4+T branch tips — and pick the root as the right-most cell along PC1. Pass these cell identifiers as tips = c(root, branch.tips) to destiny's DPT with w_width = 0.1, so the diffusion pseudotime is anchored at the root and the tips. The same root and branch tips are then supplied to dandelionR::markovProbability (root_cell, terminal_state), which computes branch probabilities via the absorbing Markov chain; the column names of the returned probability columns match the tip labels. The rationale follows Palantir: an explicit start state and terminally differentiated end states are required for a well-posed absorbing Markov chain.

## Related tools

- **dandelionR** (Implements waypoint selection, absorbing Markov chain construction, branch-probability calculation and projection (markovProbability) that consume the defined root and terminal states) — github.com/tuonglab/dandelionR
- **destiny** (Computes the diffusion map and runs DPT with the tips argument set to c(root, branch.tips) to obtain anchored diffusion pseudotime)
- **miloR** (Provides the pseudobulked neighbourhood object (pb.milo) whose PCA embedding is used to locate the extreme root and branch-tip cells)

## Examples

```
dif.pse <- DPT(dm, tips = c(root, branch.tips), w_width = 0.1)
```

## Evaluation signals

- pb.milo colData contains a 'pseudotime' column after DPT, anchored at the chosen root cell
- Branch-probability columns in colData are named exactly after the terminal states ('CD8+T', 'CD4+T')
- All branch probabilities lie in [0, 1], with each pseudobulk's probabilities summing to 1 across terminals
- Pseudotime increases monotonically away from the root when plotted on the pseudobulk PCA (RdYlBu gradient)
- Chosen tips are the literal extreme cells: max/min along PC2 for the branch tips and max along PC1 for the root

## Limitations

- destiny-based diffusion maps restrict trajectory inference to bifurcation or linear topologies, so multi-furcating systems need other root/tip strategies
- Cyclic trajectories (e.g. B cell receptor editing) cannot be represented by absorbing Markov chains, making fixed absorbing terminal states inappropriate
- Fundamental discrepancies in diffusion map and KNN graph construction between R (destiny) and Python (Dandelion/Palantir) mean results differ from the original Python implementation
- dandelionR has not been validated on B cell development data, where more complex topology would challenge the two-tip definition

## Evidence

- [methods] DPT tips argument: "dif.pse <- DPT(dm, tips = c(root, branch.tips), w_width = 0.1)"
- [readme] Palantir-inspired absorbing Markov chain: "a custom implementation of absorbing markov chain for pseudotime inference, inspired based on the [palantir](https://github.com/dpeerlab/Palantir) python package"
- [abstract] dandelionR brings diffusion maps and absorbing Markov chains to R: "bringing the VDJ feature space construction and trajectory analysis using diffusion maps and absorbing Markov chains to R"
- [discussion] cyclic trajectory limitation: "This forms a cyclic trajectory, which cannot be represented by absorbing Markov chains"
- [discussion] destiny topology limitation: [evidence span withheld — anchor status: not_found]
