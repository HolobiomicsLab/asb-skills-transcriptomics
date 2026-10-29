---
name: gene-expression-baseline-trajectory-comparison
description: Use when benchmarking whether incorporating VDJ features into trajectory
  inference yields higher fate-prediction accuracy than expression-only trajectory
  analysis, e.g. on fetal immune atlas DP-to-CD4/CD8 T-cell data.
when_to_use_negative:
- When the input data lacks paired TCR/BCR sequencing (only a gene expression matrix
  is available) — there is no VDJ arm to compare against
- When the underlying biology is cyclic (e.g. B cell receptor editing) or requires
  topologies beyond bifurcation/linear — the destiny-based diffusion map and absorbing
  Markov chain cannot represent them
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: dandelionR
  role: Constructs the VDJ feature space (setupVdjPseudobulk, vdjPseudobulk), projects
    pseudotime to cells (projectPseudotimeToCell), and provides the custom absorbing
    Markov chain pseudotime implementation used in both arms
  repo: https://github.com/tuonglab/dandelionR
- name: miloR
  role: Builds the shared neighbourhood graph (buildGraph k = 30, d = 20 on X_scvi;
    makeNhoods prop = 0.3) and pseudobulks underlying both trajectories
- name: destiny
  role: Provides DiffusionMap (n_pcs = 50, n_eigs = 10) and DPT (w_width = 0.1) used
    identically for the VDJ and expression-only trajectory arms
- name: scRepertoire
  role: Loads and processes AIRR-format contigs (loadContigs, combineTCR) and merges
    TCR information with expression via combineExpression
  repo: https://github.com/ncborcherding/scRepertoire
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:0121f888ad67253c9b2316f51df4790a0e50bbeef4a8fad02b41c19c6b28e731
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tuonglab/dandelionR
  tools:
  - Dandelion
  - R
  - dandelionR
  - destiny
  - miloR
  - scRepertoire
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# gene-expression-baseline-trajectory-comparison

## Summary

Runs a matched gene-expression-only trajectory baseline (diffusion map + absorbing Markov chain on expression-derived PCs) alongside the VDJ feature space trajectory from dandelionR, to quantify how much VDJ receptor information improves CD4/CD8 fate prediction from double-positive T cells.

## When to use

Use when benchmarking whether incorporating VDJ features into trajectory inference yields higher fate-prediction accuracy than expression-only trajectory analysis, e.g. on fetal immune atlas DP-to-CD4/CD8 T-cell data.

## When NOT to use

- When the input data lacks paired TCR/BCR sequencing (only a gene expression matrix is available) — there is no VDJ arm to compare against
- When the underlying biology is cyclic (e.g. B cell receptor editing) or requires topologies beyond bifurcation/linear — the destiny-based diffusion map and absorbing Markov chain cannot represent them

## Inputs

- demo_sce SingleCellExperiment with gene expression and X_scvi embedding (fetal immune atlas lymphoid cells)
- demo_airr VDJ contigs in AIRR format, processed via scRepertoire loadContigs and combineTCR (removeNA = TRUE, removeMulti = FALSE, filterMulti = TRUE) and merged with combineExpression
- Annotated cell-type labels for CD8+T, CD4+T, ABT(ENTRY), DP(P)_T and DP(Q)_T populations used as ground truth for scoring

## Outputs

- Per-cell pseudotime and CD4/CD8 branch probabilities for both the VDJ-feature-space trajectory and the expression-only baseline
- Fate-prediction accuracy scores for both arms, written to trajectory_accuracy_comparison.csv

## How to apply

Build both trajectories on the same miloR neighbourhood graph (buildGraph with k = 30, d = 20 on the X_scvi embedding; makeNhoods prop = 0.3). For the VDJ arm, construct the feature space with vdjPseudobulk (mode_option = 'abT'), run 20-component PCA on the Feature_space assay, then infer trajectories with destiny::DiffusionMap (n_pcs = 50, n_eigs = 10) and DPT (w_width = 0.1), defining the root as the right-most pseudobulk along PC1 and CD8/CD4 branch tips as extremes along PC2, followed by markovProbability (knn = 30). For the baseline arm, run the same diffusion-map and Markov-chain machinery on expression-derived PCs without any VDJ features. Project pseudotime and branch probabilities back to cells with projectPseudotimeToCell and score predicted CD4/CD8 fate assignments against annotated cell-type labels for both runs. The rationale is that most trajectory tools rely solely on gene expression, whereas VDJ recombination determines receptor-antigen affinity during lymphocyte development, so the VDJ arm should achieve higher accuracy.

## Related tools

- **dandelionR** (Constructs the VDJ feature space (setupVdjPseudobulk, vdjPseudobulk), projects pseudotime to cells (projectPseudotimeToCell), and provides the custom absorbing Markov chain pseudotime implementation used in both arms) — https://github.com/tuonglab/dandelionR
- **miloR** (Builds the shared neighbourhood graph (buildGraph k = 30, d = 20 on X_scvi; makeNhoods prop = 0.3) and pseudobulks underlying both trajectories)
- **destiny** (Provides DiffusionMap (n_pcs = 50, n_eigs = 10) and DPT (w_width = 0.1) used identically for the VDJ and expression-only trajectory arms)
- **scRepertoire** (Loads and processes AIRR-format contigs (loadContigs, combineTCR) and merges TCR information with expression via combineExpression) — https://github.com/ncborcherding/scRepertoire

## Examples

```
milo_object <- buildGraph(milo_object, k = 30, d = 20, reduced.dim = "X_scvi"); dm <- DiffusionMap(as.matrix(feature_space), n_pcs = 50, n_eigs = 10)
```

## Evaluation signals

- The VDJ feature space trajectory achieves higher CD4/CD8 fate-prediction accuracy than the expression-only baseline, consistent with the paper's reported improvement
- Both arms share the identical miloR graph (k = 30, d = 20, prop = 0.3), diffusion-map settings (n_pcs = 50, n_eigs = 10) and markovProbability knn = 30, differing only in the feature space
- Pseudobulk counts and feature-space dimensions are sane after vdjPseudobulk (the paper's reference space had 160 V/D/J genes and 1516 pseudobulks)
- Both accuracy values are present and comparable in trajectory_accuracy_comparison.csv

## Limitations

- destiny-based diffusion maps limit trajectory inference to bifurcation or linear topologies
- Absorbing Markov chains cannot represent cyclic trajectories such as B cell receptor editing
- Fundamental discrepancies in diffusion map and KNN graph construction between R and Python packages hinder exact reproduction of the original Dandelion results (e.g. dandelionR output 17,281 cells vs original 17,234, with 17,208 shared; Lin's CCC for CD4+ probability was 0.850, below the 0.9 threshold)

## Evidence

- [intro] improved fate prediction: "most existing trajectory analysis tools rely solely on gene expression matrices"
- [intro] paper finding on improvement: "which improved the prediction accuracy of trajectories from double-positive T cells to CD4/CD8 T cells, demonstrating significant potential for future applications"
- [methods] diffusion map settings: "dm <- DiffusionMap(as.matrix(feature_space), n_pcs = 50, n_eigs = 10)"
- [methods] shared neighbourhood graph: "milo_object <- buildGraph(milo_object, k = 30, d = 20, reduced.dim = "X_scvi")"
- [results] R/Python comparison caveat: "Lin's CCC for CD4 + probability (0.850) falls below the commonly accepted threshold of 0.9"
- [readme] markov chain implementation: "a custom implementation of absorbing markov chain for pseudotime inference, inspired based on the palantir"
