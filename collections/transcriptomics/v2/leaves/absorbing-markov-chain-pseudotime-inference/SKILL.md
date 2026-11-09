---
name: absorbing-markov-chain-pseudotime-inference
description: Use when pseudobulked VDJ feature-space data (e.g. pb.milo from vdjPseudobulk
  with a 20-component PCA) needs pseudotime and branch probabilities toward defined
  terminal states in R.
when_to_use_negative:
- The trajectory is cyclic (e.g. B cell receptor editing) — absorbing Markov chains
  cannot represent cyclic trajectories
- The dataset has a topology more complex than bifurcation or linear — the destiny-based
  diffusion map limits dandelionR's trajectory inference to such topologies
- Input is single-cell gene expression without a constructed VDJ feature space — this
  skill operates on the VDJ feature space, not raw expression matrices
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: dandelionR
  role: Provides markovProbability (waypoint selection, absorbing Markov chain, probability
    calculation, projection) consolidating the Palantir-style trajectory inference
    in R
  repo: https://github.com/tuonglab/dandelionR
- name: destiny
  role: Computes the DiffusionMap (n_pcs = 50, n_eigs = 10) and DPT pseudotime that
    feed the absorbing Markov chain
- name: Dandelion
  role: Original Python implementation of the VDJ feature space trajectory workflow
    that dandelionR replicates
  repo: https://github.com/zktuong/dandelion
- name: Palantir
  role: Python package whose absorbing-Markov-chain pseudotime and branch-probability
    approach inspired dandelionR's implementation
  repo: https://github.com/dpeerlab/Palantir
- name: miloR
  role: Produces the pseudobulked neighbourhood object (vdjPseudobulk input) on which
    trajectory inference is performed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:b72ca32e0c41960a6bf465d78019831621f375e76d51c5d2b9a41418b5837ad4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tuonglab/dandelionR
  tools:
  - Dandelion
  - Palantir
  - R
  - dandelionR
  - destiny
  - miloR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# absorbing-markov-chain-pseudotime-inference

## Summary

Infers pseudotime and terminal-branch fate probabilities on the VDJ feature space using a destiny DiffusionMap/DPT for pseudotime followed by a custom absorbing Markov chain (Palantir-style) implementation consolidated in dandelionR::markovProbability.

## When to use

Use when pseudobulked VDJ feature-space data (e.g. pb.milo from vdjPseudobulk with a 20-component PCA) needs pseudotime and branch probabilities toward defined terminal states in R.

## When NOT to use

- The trajectory is cyclic (e.g. B cell receptor editing) — absorbing Markov chains cannot represent cyclic trajectories
- The dataset has a topology more complex than bifurcation or linear — the destiny-based diffusion map limits dandelionR's trajectory inference to such topologies
- Input is single-cell gene expression without a constructed VDJ feature space — this skill operates on the VDJ feature space, not raw expression matrices

## Inputs

- Pseudobulked VDJ feature space (pb.milo) from dandelionR::vdjPseudobulk, containing the Feature_space assay
- 20-component PCA embedding of the pseudobulk feature space
- Terminal state cell identifiers (branch tips, e.g. CD8+T, CD4+T) and root cell identifier selected from the PCA

## Outputs

- Diffusion pseudotime stored as a 'pseudotime' column in the pseudobulk colData
- Branch probability columns named after terminal states (e.g. 'CD8+T', 'CD4+T') in colData, with values in [0, 1]
- PCA plots of pseudotime and branch probabilities with RdYlBu gradient; annotated object saved as pb_milo_trajectory.rds

## How to apply

Extract the Feature_space assay and its 20-component PCA embedding from the pseudobulked milo object, then define a root (right-most cell along PC1) and terminal tips (top- and bottom-most cells along PC2, e.g. CD8+T and CD4+T). Compute a diffusion map with destiny::DiffusionMap using n_pcs = 50 and n_eigs = 10, then run DPT with tips = c(root, branch.tips) and w_width = 0.1 to obtain diffusion pseudotime stored as 'pseudotime' in the colData. Run dandelionR::markovProbability with the diffusion map, terminal_state, root_cell, pseudotime_key = 'pseudotime' and knn = 30; dandelionR independently implements the Palantir-style waypoint selection, absorbing Markov chain construction, probability calculation and projection, returning branch-probability columns named after the terminal states. The rationale is that Dandelion's VDJ feature space improved prediction of trajectories from double-positive T cells to CD4/CD8 T cells, and the absorbing Markov chain formalism (inspired by Palantir) propagates transition probabilities from root to terminal states.

## Related tools

- **dandelionR** (Provides markovProbability (waypoint selection, absorbing Markov chain, probability calculation, projection) consolidating the Palantir-style trajectory inference in R) — https://github.com/tuonglab/dandelionR
- **destiny** (Computes the DiffusionMap (n_pcs = 50, n_eigs = 10) and DPT pseudotime that feed the absorbing Markov chain)
- **Dandelion** (Original Python implementation of the VDJ feature space trajectory workflow that dandelionR replicates) — https://github.com/zktuong/dandelion
- **Palantir** (Python package whose absorbing-Markov-chain pseudotime and branch-probability approach inspired dandelionR's implementation) — https://github.com/dpeerlab/Palantir
- **miloR** (Produces the pseudobulked neighbourhood object (vdjPseudobulk input) on which trajectory inference is performed)

## Examples

```
dm <- DiffusionMap(as.matrix(feature_space), n_pcs = 50, n_eigs = 10); dif.pse <- DPT(dm, tips = c(root, branch.tips), w_width = 0.1); pb.milo <- markovProbability(pb.milo, diffusionmap = dm, terminal_state = branch.tips, root_cell = root, pseudotime_key = 'pseudotime', knn = 30)
```

## Evaluation signals

- pb.milo colData contains a 'pseudotime' column populated by the destiny DPT run
- colData contains branch-probability columns named after the terminal states ('CD8+T', 'CD4+T')
- All branch probabilities lie within [0, 1]
- Probability columns exactly match the terminal-state identifiers supplied to markovProbability (terminal_state, pseudotime_key = 'pseudotime', knn = 30)
- Pseudotime and probabilities visualise sensibly on the pseudobulk PCA (RdYlBu gradient from root toward branch tips)

## Limitations

- The destiny-based diffusion map is only suitable for datasets with bifurcation or linear topology, limiting dandelionR's trajectory inference versatility
- Cyclic trajectories (e.g. B cell receptor editing) cannot be represented by absorbing Markov chains; dandelionR has not yet been applied to B cell development data
- Fundamental discrepancies in diffusion map and KNN graph construction between R and Python packages hinder full reimplementation of the original Dandelion; Lin's CCC for CD4+ probability was 0.850, below the accepted 0.9 threshold

## Evidence

- [methods] waypoint selection, absorbing Markov chain construction, probability calculation, and projection independently, consolidating them into the function dandelionR::markovProbability: [evidence span withheld — anchor status: not_found]
- [methods] dm <- DiffusionMap(as.matrix(feature_space), n_pcs = 50, n_eigs = 10): "dm <- DiffusionMap(as.matrix(feature_space), n_pcs = 50, n_eigs = 10)"
- [methods] dif.pse <- DPT(dm, tips = c(root, branch.tips), w_width = 0.1): "dif.pse <- DPT(dm, tips = c(root, branch.tips), w_width = 0.1)"
- [readme] a custom implementation of absorbing markov chain for pseudotime inference, inspired based on the palantir: "a custom implementation of absorbing markov chain for pseudotime inference, inspired based on the [palantir]"
- [discussion] destiny, is only suitable for a dataset with a bifurcation or linear topology, which limits the versatility of dandelionR: "destiny, is only suitable for a dataset with a bifurcation or linear topology, which limits the versatility of dandelionR’s trajectory inference function"
- [discussion] This forms a cyclic trajectory, which cannot be represented by absorbing Markov chains: "This forms a cyclic trajectory, which cannot be represented by absorbing Markov chains"
