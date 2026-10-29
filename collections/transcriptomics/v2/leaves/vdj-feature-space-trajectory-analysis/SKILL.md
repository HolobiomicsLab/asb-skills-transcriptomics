---
name: vdj-feature-space-trajectory-analysis
description: Use when a SingleCellExperiment with a miloR-pseudobulked VDJ feature
  space (e.g. pb.milo from vdjPseudobulk, mode_option='abT') needs pseudotime and
  CD4/CD8 branching probabilities projected back to cells.
when_to_use_negative:
- The dataset has a cyclic trajectory (e.g. B cell receptor editing), which cannot
  be represented by absorbing Markov chains.
- The data topology is more complex than a bifurcation or linear trajectory, since
  the destiny-based diffusion map limits inference to bifurcation or linear topologies.
- The input lacks a constructed VDJ feature space (no vdjPseudobulk Feature_space
  assay) and only a gene-expression matrix is available — that is the weaker baseline
  this skill is meant to improve upon.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: dandelionR
  role: R implementation providing markovProbability (absorbing Markov chain branch
    probabilities) and projectPseudotimeToCell on the VDJ feature space
  repo: https://github.com/tuonglab/dandelionR
- name: destiny
  role: computes the diffusion map (DiffusionMap, n_pcs = 50, n_eigs = 10) and diffusion
    pseudotime (DPT, w_width = 0.1) on the feature space
- name: miloR
  role: builds the neighbourhood graph and pseudobulks used to construct the VDJ feature
    space trajectory inputs
- name: Dandelion
  role: original Python implementation of the VDJ feature space trajectory workflow
    that dandelionR replicates; benchmarking reference
  repo: https://www.github.com/zktuong/dandelion
- name: Palantir
  role: Python package inspiring the absorbing Markov chain implementation; source
    of pseudotime inputs for Benchmark 2 concordance testing
  repo: https://github.com/dpeerlab/Palantir
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:2c3b76a4e64adcaed01cf513a2fe79a9dfe5db4a4107c31953fd39944ac5d4e4
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# vdj-feature-space-trajectory-analysis

## Summary

Trajectory inference on a VDJ feature space in R, replicating Dandelion/Palantir's workflow by computing diffusion maps (destiny), diffusion pseudotime (DPT), and branch probabilities via an absorbing Markov chain (dandelionR::markovProbability) on pseudobulked VDJ features.

## When to use

Use when a SingleCellExperiment with a miloR-pseudobulked VDJ feature space (e.g. pb.milo from vdjPseudobulk, mode_option='abT') needs pseudotime and CD4/CD8 branching probabilities projected back to cells.

## When NOT to use

- The dataset has a cyclic trajectory (e.g. B cell receptor editing), which cannot be represented by absorbing Markov chains.
- The data topology is more complex than a bifurcation or linear trajectory, since the destiny-based diffusion map limits inference to bifurcation or linear topologies.
- The input lacks a constructed VDJ feature space (no vdjPseudobulk Feature_space assay) and only a gene-expression matrix is available — that is the weaker baseline this skill is meant to improve upon.

## Inputs

- pseudobulked miloR object (pb.milo) with a Feature_space VDJ assay and 20-component PCA embedding, produced by vdjPseudobulk
- root and terminal-state cell identifiers (branch tips) selected from pseudobulk PCA extremes (PC1/PC2)
- optionally, Palantir-derived pseudotime for concordance benchmarking on identical inputs (sce_vdj)

## Outputs

- pb.milo colData annotated with a 'pseudotime' column and branch-probability columns ('CD8+T', 'CD4+T') with values in [0, 1]
- cell-level pseudotime and branch probabilities via projectPseudotimeToCell
- annotated object saved as pb_milo_trajectory.rds

## How to apply

Extract the Feature_space assay from the pb.milo pseudobulk object (built by vdjPseudobulk) and its 20-component PCA embedding. Define terminal states and root from PCA extremes: top-most and bottom-most pseudobulks along PC2 as the CD8+T and CD4+T branch tips, and the right-most along PC1 as the root. Compute a diffusion map with destiny::DiffusionMap (n_pcs = 50, n_eigs = 10) and run DPT with tips = c(root, branch.tips) and w_width = 0.1, storing diffusion pseudotime as 'pseudotime' in colData. Then run dandelionR::markovProbability (diffusionmap, terminal_state = branch tips, root_cell, pseudotime_key = 'pseudotime', knn = 30), which consolidates waypoint selection, absorbing Markov chain construction, probability calculation, and projection, returning branch-probability columns named after the terminal states ('CD8+T', 'CD4+T'). Finally project pseudotime and branch probabilities to individual cells with dandelionR::projectPseudotimeToCell and visualise on the pseudobulk PCA with an RdYlBu gradient.

## Related tools

- **dandelionR** (R implementation providing markovProbability (absorbing Markov chain branch probabilities) and projectPseudotimeToCell on the VDJ feature space) — https://github.com/tuonglab/dandelionR
- **destiny** (computes the diffusion map (DiffusionMap, n_pcs = 50, n_eigs = 10) and diffusion pseudotime (DPT, w_width = 0.1) on the feature space)
- **miloR** (builds the neighbourhood graph and pseudobulks used to construct the VDJ feature space trajectory inputs)
- **Dandelion** (original Python implementation of the VDJ feature space trajectory workflow that dandelionR replicates; benchmarking reference) — https://www.github.com/zktuong/dandelion
- **Palantir** (Python package inspiring the absorbing Markov chain implementation; source of pseudotime inputs for Benchmark 2 concordance testing) — https://github.com/dpeerlab/Palantir

## Examples

```
dm <- DiffusionMap(as.matrix(feature_space), n_pcs = 50, n_eigs = 10); dif.pse <- DPT(dm, tips = c(root, branch.tips), w_width = 0.1); pb.milo <- markovProbability(pb.milo, diffusionmap = dm, terminal_state = branch.tips, root_cell = root, pseudotime_key = 'pseudotime', knn = 30)
```

## Evaluation signals

- Verify pb.milo colData contains a 'pseudotime' column and branch-probability columns named after the terminal states ('CD8+T', 'CD4+T') with probabilities in [0, 1].
- Benchmark 2 concordance: on identical diffusion map and pseudotime inputs from Palantir, Lin's CCC should improve over the full Python pipeline — Kendall rank correlation rose from 0.790 to 0.925.
- Trajectory accuracy check: VDJ feature space trajectory should achieve higher CD4/CD8 fate prediction accuracy against annotated cell-type labels than the gene-expression-only baseline (trajectory_accuracy_comparison.csv).
- Diffusion map must be computed with n_pcs = 50, n_eigs = 10 and DPT with w_width = 0.1 and tips = c(root, branch.tips); markovProbability with knn = 30.

## Limitations

- The destiny-based diffusion map limits trajectory inference to bifurcation or linear topologies.
- dandelionR has not been applied to B cell development data, which involves cyclic processes that absorbing Markov chains cannot represent.
- Fundamental discrepancies in diffusion map and KNN graph construction between R and Python packages hinder full reimplementation of the original Dandelion.
- Lin's CCC for CD4+ probability was 0.850, below the commonly accepted threshold of 0.9.

## Evidence

- [methods] dandelionR computes the diffusion map and pseudotime with the destiny package, then implements waypoint selection, absorbing Markov chain construction, probability calculation, and projection independently: [evidence span withheld — anchor status: not_found]
- [discussion] destiny-based diffusion map limits trajectory inference: [evidence span withheld — anchor status: not_found]
- [abstract] R package abstract on trajectory analysis: "we present dandelionR , an R implementation of Dandelion 's trajectory analysis workflow, bringing the VDJ feature space construction and trajectory analysis using diffusion maps and absorbing Markov"
- [readme] README on custom absorbing Markov chain: "It provides all the necessary tools to interface with [`scRepertoire`](https://github.com/ncborcherding/scRepertoire) and a custom implementation of absorbing markov chain for pseudotime inference"
- [results] Kendall concordance improvement on identical inputs: [evidence span withheld — anchor status: not_found]
