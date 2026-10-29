---
name: t-cell-differentiation-pseudotime-visualization
description: Use when single-cell VDJ-seq pseudobulk data (e.g. miloR pseudobulks
  over an abT VDJ feature space) must be ordered along an alpha-beta T-cell differentiation
  trajectory with terminal-state branch probabilities.
when_to_use_negative:
- The trajectory is cyclic (e.g. B cell receptor editing) — cyclic trajectories cannot
  be represented by absorbing Markov chains
- The data has a topology more complex than a bifurcation or linear trajectory — the
  destiny-based diffusion map limits dandelionR's trajectory inference to bifurcation
  or linear topologies
- The goal is B cell development analysis — dandelionR has not yet been applied to
  B cell development data
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: destiny
  role: Computes the diffusion map (DiffusionMap, n_pcs = 50, n_eigs = 10) and DPT
    diffusion pseudotime on the VDJ feature space
- name: dandelionR
  role: Provides markovProbability, which independently implements waypoint selection,
    absorbing Markov chain construction, probability calculation, and projection
  repo: github.com/tuonglab/dandelionR
- name: miloR
  role: Produces the pseudobulked pb.milo object (via vdjPseudobulk) whose Feature_space
    assay and PCA embedding feed the trajectory step
- name: Dandelion
  role: Original Python implementation of the VDJ feature space trajectory workflow
    that dandelionR replicates; comparison reference for branch probabilities (Lin's
    CCC for CD4+ probability was 0.850)
  repo: github.com/zktuong/dandelion
- name: Palantir
  role: Inspiration for dandelionR's absorbing Markov chain pseudotime inference approach
  repo: dpeerlab/Palantir
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:13c2194f75c08edb3c44f4f7b0743873d16f1b3694cdc5ec215831fb6908d94d
schema_version: 0.2.0
metadata:
  license_tier: open
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
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# t-cell-differentiation-pseudotime-visualization

## Summary

Reconstructs T-cell differentiation trajectories on the VDJ feature space by computing a diffusion map and diffusion pseudotime with the destiny R package, then computing branch absorption probabilities with dandelionR::markovProbability (a Palantir-style absorbing Markov chain) and visualising them on the pseudobulk PCA.

## When to use

Use when single-cell VDJ-seq pseudobulk data (e.g. miloR pseudobulks over an abT VDJ feature space) must be ordered along an alpha-beta T-cell differentiation trajectory with terminal-state branch probabilities.

## When NOT to use

- The trajectory is cyclic (e.g. B cell receptor editing) — cyclic trajectories cannot be represented by absorbing Markov chains
- The data has a topology more complex than a bifurcation or linear trajectory — the destiny-based diffusion map limits dandelionR's trajectory inference to bifurcation or linear topologies
- The goal is B cell development analysis — dandelionR has not yet been applied to B cell development data

## Inputs

- Pseudobulked miloR object (pb.milo) containing a Feature_space assay built from V/D/J gene usage and a 20-component PCA embedding
- Terminal state and root cell identifiers selected from the pseudobulk PCA (CD8+T and CD4+T branch tips; PC1 root)

## Outputs

- pb.milo object with 'pseudotime' in colData and branch-probability columns named after the terminal states ('CD8+T', 'CD4+T')
- PCA scatter plots of pseudotime and branch probabilities with an RdYlBu colour gradient
- Saved RDS file pb_milo_trajectory.rds

## How to apply

Start from the pseudobulked VDJ feature space produced by vdjPseudobulk (pb.milo), extracting the Feature_space assay and its 20-component PCA embedding. Define the root as the right-most cell along PC1 and terminal states as the top-most and bottom-most cells along PC2 (CD8+T and CD4+T branch tips). Compute a diffusion map with destiny's DiffusionMap on the feature space using n_pcs = 50 and n_eigs = 10, then run DPT with tips = c(root, branch.tips) and w_width = 0.1 to obtain diffusion pseudotime stored as 'pseudotime' in the pseudobulk colData. Run dandelionR::markovProbability with the destiny diffusion map, the terminal branch tips, the root cell, pseudotime_key = 'pseudotime', and knn = 30 to get branch absorption probabilities. Finally, plot pseudotime and branch probabilities on the pseudobulk PCA using an RdYlBu colour gradient and save the annotated object as pb_milo_trajectory.rds.

## Related tools

- **destiny** (Computes the diffusion map (DiffusionMap, n_pcs = 50, n_eigs = 10) and DPT diffusion pseudotime on the VDJ feature space)
- **dandelionR** (Provides markovProbability, which independently implements waypoint selection, absorbing Markov chain construction, probability calculation, and projection) — github.com/tuonglab/dandelionR
- **miloR** (Produces the pseudobulked pb.milo object (via vdjPseudobulk) whose Feature_space assay and PCA embedding feed the trajectory step)
- **Dandelion** (Original Python implementation of the VDJ feature space trajectory workflow that dandelionR replicates; comparison reference for branch probabilities (Lin's CCC for CD4+ probability was 0.850)) — github.com/zktuong/dandelion
- **Palantir** (Inspiration for dandelionR's absorbing Markov chain pseudotime inference approach) — dpeerlab/Palantir

## Examples

```
dm <- DiffusionMap(as.matrix(feature_space), n_pcs = 50, n_eigs = 10); dif.pse <- DPT(dm, tips = c(root, branch.tips), w_width = 0.1); pb.milo <- markovProbability(pb.milo, diffusionmap = dm, terminal_state = branch.tips, root_cell = root, pseudotime_key = 'pseudotime', knn = 30)
```

## Evaluation signals

- The pb.milo colData contains a 'pseudotime' column and branch-probability columns named after the terminal states ('CD8+T', 'CD4+T')
- All branch probabilities lie within [0, 1]
- DPT was run with tips = c(root, branch.tips) and w_width = 0.1 on a diffusion map computed with n_pcs = 50 and n_eigs = 10
- Pseudotime increases from the PC1 root cell toward the PC2 branch tips on the pseudobulk PCA
- CD4+ branch probabilities agree with the original Python Dandelion output approximately at Lin's CCC of 0.850 (below the commonly accepted 0.9 threshold)

## Limitations

- The destiny-based diffusion map is only suitable for datasets with bifurcation or linear topology, limiting the versatility of dandelionR's trajectory inference
- Cyclic trajectories (e.g. B cell receptor editing) cannot be represented by absorbing Markov chains
- Fundamental discrepancies in diffusion map and KNN graph construction between R and Python packages hinder full reimplementation; Lin's CCC for CD4+ probability was 0.850, below the accepted threshold of 0.9
- It remains unclear whether brute-force or KD-tree KNN methods more accurately capture the underlying biological structure

## Evidence

- [methods] dandelionR computes the diffusion map and pseudotime with the destiny package: "dm <- DiffusionMap(as.matrix(feature_space), n_pcs = 50, n_eigs = 10)"
- [methods] DPT with root and branch tips and w_width 0.1 yields diffusion pseudotime: "dif.pse <- DPT(dm, tips = c(root, branch.tips), w_width = 0.1)"
- [intro] markovProbability consolidates waypoint selection, absorbing Markov chain construction, probability calculation, and projection: "a custom implementation of absorbing markov chain for pseudotime inference"
- [abstract] dandelionR brings VDJ feature space trajectory analysis using diffusion maps and absorbing Markov chains to R: "bringing the VDJ feature space construction and trajectory analysis using diffusion maps and absorbing Markov chains to R"
- [discussion] The destiny-based diffusion map limits trajectory inference to bifurcation or linear topologies: "destiny, is only suitable for a dataset with a bifurcation or linear topology, which limits the versatility of dandelionR’s trajectory inference function"
- [readme] dandelionR provides a custom absorbing Markov chain implementation inspired by Palantir: "a custom implementation of absorbing markov chain for pseudotime inference, inspired based on the [palantir](https://github.com/dpeerlab/Palantir) python package"
