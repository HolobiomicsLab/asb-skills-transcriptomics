---
name: absorbing-markov-chain-branch-probability-estimation
description: Use when a SingleCellExperiment or Milo pseudobulk object with a VDJ
  feature space and a diffusion map is available and you need branch probabilities
  into defined terminal states (e.g. CD4+T/CD8+T tips) plus diffusion pseudotime in
  R.
when_to_use_negative:
- Trajectories with cyclic topology (e.g. B cell receptor editing) — absorbing Markov
  chains cannot represent cyclic trajectories
- Datasets whose topology is not bifurcation or linear — the destiny-based diffusion
  map limits trajectory inference to such topologies
- B cell development data — dandelionR has not yet been applied to B cell development,
  which involves more complex processes than T cell development
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: dandelionR
  role: Provides markovProbability and projectPseudotimeToCell for the absorbing Markov
    chain branch probability estimation and projection to cells
  repo: https://github.com/tuonglab/dandelionR
- name: destiny
  role: Computes the DiffusionMap (n_pcs = 50, n_eigs = 10) and DPT pseudotime used
    as inputs to the Markov chain
- name: miloR
  role: Builds the neighbourhood graph and pseudobulk object (buildGraph k = 30, makeNhoods,
    vdjPseudobulk) on which markovProbability operates
- name: Palantir
  role: Reference Python implementation of the absorbing Markov chain concept used
    to benchmark concordance of dandelionR outputs
  repo: https://github.com/dpeerlab/Palantir
- name: scRepertoire
  role: Interfaces TCR contig processing (combineTCR, combineExpression) feeding the
    VDJ feature space prior to trajectory analysis
  repo: https://github.com/ncborcherding/scRepertoire
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:615484c95e8b7ee06f26399f9da01f82304cee763fbe16952c97b5376df6065d
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# absorbing-markov-chain-branch-probability-estimation

## Summary

Estimates cell-fate (branch) probabilities and pseudotime by constructing an absorbing Markov chain on a diffusion map representation, as implemented in dandelionR::markovProbability and inspired by the Palantir Python package. Used for VDJ feature-space trajectory analysis of lymphocyte differentiation (e.g. CD4+ vs CD8+ T cell branching).

## When to use

Use when a SingleCellExperiment or Milo pseudobulk object with a VDJ feature space and a diffusion map is available and you need branch probabilities into defined terminal states (e.g. CD4+T/CD8+T tips) plus diffusion pseudotime in R.

## When NOT to use

- Trajectories with cyclic topology (e.g. B cell receptor editing) — absorbing Markov chains cannot represent cyclic trajectories
- Datasets whose topology is not bifurcation or linear — the destiny-based diffusion map limits trajectory inference to such topologies
- B cell development data — dandelionR has not yet been applied to B cell development, which involves more complex processes than T cell development

## Inputs

- SingleCellExperiment with VDJ feature space (sce_vdj, Dandelion tutorial data derived from Suo et al.)
- Milo pseudobulk object from miloR (buildGraph k = 30 d = 20, makeNhoods, vdjPseudobulk)
- DiffusionMap object from destiny::DiffusionMap (n_pcs = 50, n_eigs = 10)
- Diffusion pseudotime from destiny::DPT (w_width = 0.1)
- Defined root cell and terminal branch tip identifiers (e.g. CD8+T, CD4+T pseudobulks)

## Outputs

- Branch probability columns (e.g. CD8+T, CD4+T) in pseudobulk and cell-level colData
- Cell-level pseudotime transferred via projectPseudotimeToCell
- Concordance metrics (e.g. Lin's CCC, Kendall tau, MAE) against reference Palantir outputs, e.g. benchmark2_concordance.csv

## How to apply

Compute a diffusion map on the feature space with destiny::DiffusionMap (n_pcs = 50, n_eigs = 10), then define a root cell and branch tips from pseudobulk PCA extremes (e.g. CD8+T as top-most and CD4+T as bottom-most on PC2, root as right-most on PC1) and compute pseudotime with destiny::DPT (w_width = 0.1). Run dandelionR::markovProbability on the milo pseudobulk object with terminal_state set to the branch tips, the root_cell, and knn = 30 to obtain branch probabilities. Project results back to single cells with dandelionR::projectPseudotimeToCell using value_key = c("pseudotime", "CD8+T", "CD4+T"). The rationale is that dandelionR's absorbing Markov chain implementation, inspired by Palantir, showed strong concordance: with identical diffusion map and pseudotime inputs, Kendall rank correlation against Palantir branch probabilities increased from 0.790 to 0.925, though Lin's CCC for CD4+ probability (0.850) fell below the commonly accepted threshold of 0.9.

## Related tools

- **dandelionR** (Provides markovProbability and projectPseudotimeToCell for the absorbing Markov chain branch probability estimation and projection to cells) — https://github.com/tuonglab/dandelionR
- **destiny** (Computes the DiffusionMap (n_pcs = 50, n_eigs = 10) and DPT pseudotime used as inputs to the Markov chain)
- **miloR** (Builds the neighbourhood graph and pseudobulk object (buildGraph k = 30, makeNhoods, vdjPseudobulk) on which markovProbability operates)
- **Palantir** (Reference Python implementation of the absorbing Markov chain concept used to benchmark concordance of dandelionR outputs) — https://github.com/dpeerlab/Palantir
- **scRepertoire** (Interfaces TCR contig processing (combineTCR, combineExpression) feeding the VDJ feature space prior to trajectory analysis) — https://github.com/ncborcherding/scRepertoire

## Examples

```
markovProbability(milo_object, terminal_state = c(tip1, tip2), root_cell = root, knn = 30); projectPseudotimeToCell(milo_object, sce, value_key = c("pseudotime", "CD8+T", "CD4+T"))
```

## Evaluation signals

- Lin's concordance correlation coefficient between R-derived and reference (Palantir) pseudotime and CD4+ probabilities should be reported; expect CCC close to but below 0.9 (CD4+ probability CCC = 0.850 in the benchmark)
- Kendall rank correlation of branch probabilities against Palantir should be high (0.925 when using identical diffusion map and pseudotime inputs)
- Cell-level output from projectPseudotimeToCell should contain value_key columns 'pseudotime', 'CD8+T', 'CD4+T' in colData with probabilities bounded in [0, 1]
- Pseudobulk branch tips should match the expected PCA extremes (CD8+T top-most and CD4+T bottom-most on PC2; root right-most on PC1)
- The workflow should reproduce Palantir branching probabilities at the pseudobulk level when given identical diffusion map representation and transferred pseudotime

## Limitations

- The destiny-based diffusion map limits trajectory inference to bifurcation or linear topologies, restricting versatility
- Cyclic trajectories, such as B cell receptor editing, cannot be represented by absorbing Markov chains
- Fundamental discrepancies in diffusion map and KNN graph construction between R and Python packages hinder full reimplementation of original Dandelion and Palantir results

## Evidence

- [abstract] trajectory analysis using diffusion maps and absorbing Markov chains: "bringing the VDJ feature space construction and trajectory analysis using diffusion maps and absorbing Markov chains to R"
- [methods] DiffusionMap parameters: "dm <- DiffusionMap(as.matrix(feature_space), n_pcs = 50, n_eigs = 10)"
- [methods] DPT parameters: "dif.pse <- DPT(dm, tips = c(root, branch.tips), w_width = 0.1)"
- [readme] custom absorbing Markov chain implementation inspired by Palantir: "a custom implementation of absorbing markov chain for pseudotime inference, inspired based on the [palantir](https://github.com/dpeerlab/Palantir) python package"
- [results] Benchmark 2 concordance result: [evidence span withheld — anchor status: not_found]
- [results] Kendall correlation improvement: [evidence span withheld — anchor status: not_found]
