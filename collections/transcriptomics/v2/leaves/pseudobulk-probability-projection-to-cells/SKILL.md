---
name: pseudobulk-probability-projection-to-cells
description: Use when trajectory inference was performed on VDJ pseudobulks (e.g.
  miloR neighbourhoods from the Suo et al. fetal immune atlas data) and you need per-cell
  pseudotime and CD4/CD8 branch probabilities to score predicted cell fates against
  annotated labels.
when_to_use_negative:
- The trajectory is cyclic (e.g. B cell receptor editing) and cannot be represented
  by absorbing Markov chains, so branch probabilities to project are undefined
- The dataset topology is not bifurcation or linear — destiny-based diffusion map
  inference underpinning the probabilities is unsuitable
- Input is already per-cell pseudotime/fate data and no pseudobulk aggregation was
  performed
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: dandelionR
  role: Provides projectPseudotimeToCell to project pseudobulk pseudotime and branch
    probabilities back to individual cells
  repo: https://github.com/tuonglab/dandelionR
- name: destiny
  role: Diffusion map and DPT used to compute the pseudobulk pseudotime that is projected
    to cells
- name: miloR
  role: Builds the neighbourhood graph and pseudobulks whose values are projected
    back to cells
- name: scRepertoire
  role: Loads and combines TCR contigs and merges VDJ information onto the SingleCellExperiment
    prior to pseudobulking
  repo: https://github.com/ncborcherding/scRepertoire
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:a48b8475d0610ea02b7931f907f83c3431ae2e87f84326243a87e52f302e3563
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# pseudobulk-probability-projection-to-cells

## Summary

Projects pseudobulk-level trajectory results (pseudotime and branch/fate probabilities computed on the VDJ feature space via diffusion maps and absorbing Markov chains) back to individual cells using dandelionR's projectPseudotimeToCell, enabling cell-level fate scoring in the ab T-cell DP-to-CD4/CD8 analysis.

## When to use

Use when trajectory inference was performed on VDJ pseudobulks (e.g. miloR neighbourhoods from the Suo et al. fetal immune atlas data) and you need per-cell pseudotime and CD4/CD8 branch probabilities to score predicted cell fates against annotated labels.

## When NOT to use

- The trajectory is cyclic (e.g. B cell receptor editing) and cannot be represented by absorbing Markov chains, so branch probabilities to project are undefined
- The dataset topology is not bifurcation or linear — destiny-based diffusion map inference underpinning the probabilities is unsuitable
- Input is already per-cell pseudotime/fate data and no pseudobulk aggregation was performed

## Inputs

- SingleCellExperiment (sce_vdj / demo_sce) with VDJ feature space pseudobulk results, DiffusionMap/DPT pseudotime, and markovProbability branch probabilities
- Per-cell annotation labels (e.g. CD8+T, CD4+T, ABT(ENTRY), DP(P)_T, DP(Q)_T) for fate-scoring validation
- miloR neighbourhood-to-cell membership structure used by vdjPseudobulk

## Outputs

- Per-cell pseudotime values projected from pseudobulk neighbourhoods
- Per-cell CD4/CD8 branch probability assignments
- Predicted fate labels per cell and accuracy comparison (e.g. trajectory_accuracy_comparison.csv) against annotated cell types for VDJ-based vs expression-only runs

## How to apply

First compute pseudobulk-level results: build the VDJ feature space with vdjPseudobulk (mode_option = 'abT') on a miloR object created with buildGraph (k = 30, d = 20 on X_scvi) and makeNhoods (prop = 0.3), run a 20-component PCA on the Feature_space assay, then infer pseudotime and branch probabilities with destiny::DiffusionMap (n_pcs = 50, n_eigs = 10), DPT (w_width = 0.1, root at the right-most pseudobulk along PC1, CD8/CD4 tips as PC2 extremes) and markovProbability (knn = 30). Then call projectPseudotimeToCell to transfer pseudotime and the CD4/CD8 branch probabilities from each pseudobulk neighbourhood back to the member cells of the SingleCellExperiment. Finally, assign each cell its higher-probability fate (CD4+ vs CD8+) and score against annotated cell-type labels, comparing VDJ-based accuracy against a gene-expression-only baseline; the article reports this projection step as one of the result-projecting functions dandelionR reproduces from the original Dandelion Python software.

## Related tools

- **dandelionR** (Provides projectPseudotimeToCell to project pseudobulk pseudotime and branch probabilities back to individual cells) — https://github.com/tuonglab/dandelionR
- **destiny** (Diffusion map and DPT used to compute the pseudobulk pseudotime that is projected to cells)
- **miloR** (Builds the neighbourhood graph and pseudobulks whose values are projected back to cells)
- **scRepertoire** (Loads and combines TCR contigs and merges VDJ information onto the SingleCellExperiment prior to pseudobulking) — https://github.com/ncborcherding/scRepertoire

## Examples

```
sce <- projectPseudotimeToCell(pb.milo, sce, dm, dif.pse, markov_probs)
```

## Evaluation signals

- Projected per-cell fate assignments should yield higher CD4/CD8 prediction accuracy for the VDJ feature space run than for the matched gene-expression-only baseline, matching the paper's reported improvement
- Every cell in the target populations (CD8+T, CD4+T, ABT(ENTRY), DP(P)_T, DP(Q)_T) receives a non-NA pseudotime and branch probability after projection
- Projected pseudotime should increase monotonically with distance from the DP(ENTRY)-side root along the diffusion map, consistent with the right-most-PC1 root definition
- Branch probabilities per cell should sum to 1 across CD4/CD8 fates (absorbing Markov chain property)
- Dandelion-vs-dandelionR concordance check: the R output retained 17,281 cells vs 17,234 in original Dandelion with 17,208 shared, so projected results should cover the shared cell set

## Limitations

- destiny-based diffusion map limits trajectory inference to bifurcation or linear topologies, restricting the trajectories whose probabilities can be projected
- Absorbing Markov chains cannot represent cyclic trajectories (e.g. B cell receptor editing), so the projection framework does not extend to such processes
- Fundamental discrepancies in diffusion map and KNN graph construction between R and Python packages hinder exact equivalence; Lin's CCC for CD4+ probability was 0.850, below the commonly accepted threshold of 0.9
- dandelionR has not yet been applied to B cell development data

## Evidence

- [methods] dandelionR reproduces the preprocessing, feature-space-building and result-projecting functions of the original Dandelion software: "As an R implementation of Dandelion, it aims to reproduce the preprocessing, feature-space-building and result-projecting functions of the original software."
- [abstract] diffusion maps and absorbing Markov chains brought to R: "we present dandelionR , an R implementation of Dandelion 's trajectory analysis workflow, bringing the VDJ feature space construction and trajectory analysis using diffusion maps and absorbing Markov"
- [intro] Dandelion's VDJ feature space improved DP to CD4/CD8 trajectory prediction: "which improved the prediction accuracy of trajectories from double-positive T cells to CD4/CD8 T cells, demonstrating significant potential for future applications"
- [discussion] destiny-based diffusion map topology limitation: "destiny, is only suitable for a dataset with a bifurcation or linear topology, which limits the versatility of dandelionR’s trajectory inference function"
- [readme] README: dandelionR interfaces with scRepertoire and implements absorbing Markov chain pseudotime: "It provides all the necessary tools to interface with [`scRepertoire`](https://github.com/ncborcherding/scRepertoire) and a custom implementation of absorbing markov chain for pseudotime inference"
