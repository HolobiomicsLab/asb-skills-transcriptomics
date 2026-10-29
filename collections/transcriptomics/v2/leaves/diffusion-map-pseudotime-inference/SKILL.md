---
name: diffusion-map-pseudotime-inference
description: Use when a pseudobulked VDJ feature space (e.g. miloR pseudobulks from
  vdjPseudobulk) needs pseudotime and terminal-branch probabilities computed via diffusion
  maps and an absorbing Markov chain in R.
when_to_use_negative:
- The dataset has a cyclic topology (e.g. B cell receptor editing), which cannot be
  represented by absorbing Markov chains
- The data topology is neither bifurcating nor linear — the destiny-based diffusion
  map limits dandelionR's trajectory inference to bifurcation or linear topologies
- Only gene-expression data is available without a constructed VDJ feature space;
  this skill operates on the pseudobulked VDJ feature space, not the transcriptomic
  matrix
edam_operation: http://edamontology.org/operation_0327
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: destiny
  role: Computes the diffusion map (DiffusionMap, n_pcs = 50, n_eigs = 10) and diffusion
    pseudotime (DPT, w_width = 0.1) on the VDJ feature space
- name: dandelionR
  role: Provides markovProbability (absorbing Markov chain branch probabilities, knn
    = 30) and projectPseudotimeToCell to transfer pseudobulk values to single cells
  repo: https://github.com/tuonglab/dandelionR
- name: miloR
  role: Supplies the pseudobulk neighbourhood object (pb.milo) whose Feature_space
    and colData the trajectory analysis operates on
- name: Palantir
  role: Python reference implementation whose branching probability outputs are used
    as the comparison target for concordance benchmarks
  repo: https://github.com/dpeerlab/Palantir
provenance:
  source_task_ids:
  - task_006
  - task_003
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:5c86d7844eec0e6f8d3229a9320a1cb1407859b6fc69e7bac5859ffc9c47a673
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
  - scRepertoire
  edam_operation: http://edamontology.org/operation_0327
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# diffusion-map-pseudotime-inference

## Summary

Infers pseudotime and branch probabilities on the VDJ feature space using destiny diffusion maps, DPT, and dandelionR's absorbing Markov chain (markovProbability), replicating the Palantir-style trajectory workflow of Dandelion in R.

## When to use

Use when a pseudobulked VDJ feature space (e.g. miloR pseudobulks from vdjPseudobulk) needs pseudotime and terminal-branch probabilities computed via diffusion maps and an absorbing Markov chain in R.

## When NOT to use

- The dataset has a cyclic topology (e.g. B cell receptor editing), which cannot be represented by absorbing Markov chains
- The data topology is neither bifurcating nor linear — the destiny-based diffusion map limits dandelionR's trajectory inference to bifurcation or linear topologies
- Only gene-expression data is available without a constructed VDJ feature space; this skill operates on the pseudobulked VDJ feature space, not the transcriptomic matrix

## Inputs

- Pseudobulked VDJ feature space (pb.milo from vdjPseudobulk) with a Feature_space assay and 20-component PCA embedding
- sce_vdj / Suo et al. fetal immune T-cell data converted to SingleCellExperiment
- Optional: Palantir-derived diffusion map representation and pseudotime for concordance benchmarking

## Outputs

- pseudotime column in pseudobulk colData (from DPT, w_width = 0.1)
- Branch-probability columns named after terminal states ('CD8+T', 'CD4+T') in pseudobulk colData, in [0, 1]
- Per-cell projected pseudotime and branch probabilities via projectPseudotimeToCell
- Concordance metrics (e.g. benchmark2_concordance.csv with Lin's CCC, Kendall tau, Pearson r, MAE)

## How to apply

Extract the Feature_space assay from the pseudobulk object (pb.milo) and define terminal states and root from its 20-component PCA: the top-most and bottom-most cells along PC2 as CD8+T and CD4+T branch tips, and the right-most cell along PC1 as the root. Compute a diffusion map with destiny::DiffusionMap using n_pcs = 50 and n_eigs = 10, then run destiny::DPT with tips = c(root, branch.tips) and w_width = 0.1 to obtain pseudotime stored as 'pseudotime' in the pseudobulk colData. Run dandelionR::markovProbability with the destiny diffusion map, terminal_state = branch tips, root_cell, pseudotime_key = 'pseudotime', and knn = 30; this consolidates waypoint selection, absorbing Markov chain construction, and probability calculation, returning branch-probability columns named after the terminal states ('CD8+T', 'CD4+T'). Project pseudobulk values back to individual cells with dandelionR::projectPseudotimeToCell using value_key = c('pseudotime', 'CD8+T', 'CD4+T'). When reproducing Palantir on identical inputs, the same diffusion map and transferred pseudotime improve agreement (Kendall rank correlation 0.790 to 0.925), though Lin's CCC of 0.850 for CD4+ probability remains below the 0.9 acceptance threshold.

## Related tools

- **destiny** (Computes the diffusion map (DiffusionMap, n_pcs = 50, n_eigs = 10) and diffusion pseudotime (DPT, w_width = 0.1) on the VDJ feature space)
- **dandelionR** (Provides markovProbability (absorbing Markov chain branch probabilities, knn = 30) and projectPseudotimeToCell to transfer pseudobulk values to single cells) — https://github.com/tuonglab/dandelionR
- **miloR** (Supplies the pseudobulk neighbourhood object (pb.milo) whose Feature_space and colData the trajectory analysis operates on)
- **Palantir** (Python reference implementation whose branching probability outputs are used as the comparison target for concordance benchmarks) — https://github.com/dpeerlab/Palantir

## Examples

```
dm <- DiffusionMap(as.matrix(feature_space), n_pcs = 50, n_eigs = 10); dif.pse <- DPT(dm, tips = c(root, branch.tips), w_width = 0.1); pb.milo <- markovProbability(pb.milo, diffusionmap = dm, terminal_state = c('CD8+T','CD4+T'), root_cell = root, pseudotime_key = 'pseudotime', knn = 30)
```

## Evaluation signals

- pb.milo colData contains a 'pseudotime' column and branch-probability columns named exactly after the terminal states ('CD8+T', 'CD4+T')
- All branch probabilities lie within [0, 1]
- Benchmark 2 concordance on identical inputs: Kendall rank correlation increases from 0.790 to 0.925 vs Palantir; Lin's CCC for CD4+ probability ~0.850
- Branch tips and root are reproducibly recovered from PCA extremes: CD8+T/CD4+T as top-most/bottom-most on PC2, root as right-most on PC1
- Per-cell projection via projectPseudotimeToCell returns values for value_key = c('pseudotime', 'CD8+T', 'CD4+T')

## Limitations

- The destiny-based diffusion map is only suitable for datasets with a bifurcation or linear topology, limiting dandelionR's trajectory inference versatility
- dandelionR has not yet been applied to B cell development data, and cyclic trajectories (e.g. B cell receptor editing) cannot be represented by absorbing Markov chains
- Fundamental discrepancies in diffusion map construction and KNN graph construction between R and Python packages hinder full reimplementation, causing seed-handling discrepancies vs Palantir

## Evidence

- [methods] Compute a diffusion map on the feature space with the destiny R package using n_pcs = 50 and n_eigs = 10, then run DPT with tips = c(root, branch.tips) and w_width = 0.1: "dm <- DiffusionMap(as.matrix(feature_space), n_pcs = 50, n_eigs = 10)"
- [methods] dif.pse <- DPT(dm, tips = c(root, branch.tips), w_width = 0.1): "dif.pse <- DPT(dm, tips = c(root, branch.tips), w_width = 0.1)"
- [full_text] markovProbability consolidates waypoint selection, absorbing Markov chain construction, and probability calculation: [evidence span withheld — anchor status: not_found]
- [results] Kendall rank correlation increases from 0.790 to 0.925 on identical inputs: [evidence span withheld — anchor status: not_found]
- [discussion] destiny-based diffusion map limits topology: "destiny, is only suitable for a dataset with a bifurcation or linear topology, which limits the versatility of dandelionR’s trajectory inference function"
- [readme] absorbing markov chain for pseudotime inference inspired by palantir: "It provides all the necessary tools to interface with [`scRepertoire`](https://github.com/ncborcherding/scRepertoire) and a custom implementation of absorbing markov chain for pseudotime inference,"
