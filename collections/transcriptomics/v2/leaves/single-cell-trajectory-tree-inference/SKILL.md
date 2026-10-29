---
name: single-cell-trajectory-tree-inference
description: Use when you have a multi-sample scRNA-seq dataset with per-cell sample
  annotation and a low-dimensional embedding (e.
when_to_use_negative:
- Your trajectory was built with Monocle2, Monocle3, or Slingshot — Lamian modules
  1 and 2 do not support these trajectories, and tailoring them is deferred to future
  work
- You only have a single biological sample (no per-sample annotation or replicates)
  — differential topology testing requires multiple samples per condition and does
  not account for cross-sample variability otherwise
- You need a systematic comparison of harmonization/embedding methods before tree
  inference — the article states this is explicitly beyond the study's scope
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Lamian
  role: R package providing infer_tree_structure, evaluate_uncertainty, plotmclust,
    and branchPropTest for tree construction, uncertainty quantification, and differential
    topology testing
  repo: https://github.com/Winnie09/Lamian
- name: TSCAN
  role: Cluster-based minimum spanning tree method used by Lamian as the default pseudotime
    trajectory constructor; the updated version must be installed from GitHub
  repo: github.com/zji90/TSCAN
- name: scRNA-seq
  role: 'Input data modality: multi-sample single-cell RNA-sequencing counts from
    which the pseudotemporal tree is inferred'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:479dd50b21de58519ceaf6049bbc3543647ef93a421ccc063e0f590d92c0d62a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - TSCAN
  - branchPropTest
  - infer_tree_structure
  - plotmclust
  - scRNA-seq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# single-cell-trajectory-tree-inference

## Summary

Infer a cluster-based minimum spanning tree pseudotime trajectory from multi-sample scRNA-seq data using Lamian's infer_tree_structure (TSCAN-based), visualize it with plotmclust, and quantify branch-level uncertainty via bootstrap detection rates, optionally followed by binomial logistic-regression differential topology testing (branchPropTest) across sample covariates.

## When to use

Use when you have a multi-sample scRNA-seq dataset with per-cell sample annotation and a low-dimensional embedding (e.g. PCA) and need to reconstruct a branching pseudotime tree, assess how reliably each branch is recovered, or test whether tree topology differs across a sample covariate such as sex.

## When NOT to use

- Your trajectory was built with Monocle2, Monocle3, or Slingshot — Lamian modules 1 and 2 do not support these trajectories, and tailoring them is deferred to future work
- You only have a single biological sample (no per-sample annotation or replicates) — differential topology testing requires multiple samples per condition and does not account for cross-sample variability otherwise
- You need a systematic comparison of harmonization/embedding methods before tree inference — the article states this is explicitly beyond the study's scope

## Inputs

- Gene-by-cell expression matrix (e.g. hca_bm_saver raw/SAVER-corrected counts)
- Low-dimensional cell embedding (PCA matrix, e.g. hca_bm_pca)
- Cell-to-sample annotation data frame (e.g. hca_bm_cellanno)
- Covariate/design data frame keyed by sample (e.g. sex per donor) for differential topology testing

## Outputs

- Cluster-based MST tree structure with branch connectivity (e.g. 2 --> 3 --> 1, 2 --> 4, 2 --> 5)
- Per-branch bootstrap detection rates plus mean/SD cell proportions per sample
- Tree visualization (e.g. tree_structure.png from plotmclust)
- Per-branch differential topology p-values from binomial logistic regression (branchPropTest)

## How to apply

Load the gene-by-cell expression matrix, PCA embedding, and cell-to-sample annotation (e.g. the hca_bm_saver / hca_bm_pca / hca_bm_cellanno objects shipped with Lamian). Infer the tree with infer_tree_structure on the PCA embedding; for the HCA bone marrow example set origin.marker = 'CD34' and number.cluster = 5, which yields a three-branch tree (2 --> 3 --> 1, 2 --> 4, 2 --> 5). Visualize with plotmclust (cell_point_size = 0.5, axes labeled 'Principal component 1/2') to sanity-check the topology against known marker biology. Quantify branch uncertainty with evaluate_uncertainty, matching bootstrapped branches to original branches using Jaccard statistics and overlap coefficients over bootstrap replicates to obtain each branch's detection rate (fraction of bootstrap settings with a match) plus per-branch sample proportion means and standard deviations. If a covariate is present (e.g. sex = c(0, rep(1, 4), rep(0, 3)) for the 8-donor HCA-BM design), run branchPropTest to fit a binomial logistic regression of each branch's cell proportion on that covariate; the rationale is that TSCAN is Lamian's default trajectory builder for its flexibility and scalability, and bootstrap detection rates separate structurally robust branches from topology-inference uncertainty before any differential comparison.

## Related tools

- **Lamian** (R package providing infer_tree_structure, evaluate_uncertainty, plotmclust, and branchPropTest for tree construction, uncertainty quantification, and differential topology testing) — https://github.com/Winnie09/Lamian
- **TSCAN** (Cluster-based minimum spanning tree method used by Lamian as the default pseudotime trajectory constructor; the updated version must be installed from GitHub) — github.com/zji90/TSCAN
- **scRNA-seq** (Input data modality: multi-sample single-cell RNA-sequencing counts from which the pseudotemporal tree is inferred)

## Examples

```
res <- Lamian::infer_tree_structure(pca = man_tree_data[['pca']], origin.marker = 'CD34', number.cluster = 5); result <- evaluate_uncertainty(res); plotmclust(res, cell_point_size = 0.5)
```

## Evaluation signals

- On the HCA-BM example (origin.marker='CD34', number.cluster=5), the inferred tree matches the expected three-branch structure 2 --> 3 --> 1, 2 --> 4, 2 --> 5
- Branch detection rates reproduce the reported values (~0.938, 0.953, 0.615) and branch cell proportions (~41.1%, 48.4%, 10.5%)
- branchPropTest sex p-values reproduce the reported ~0.35, 0.64, 0.94, supporting no significant sex-associated topology change
- plotmclust output shows cells colored along the tree with axes labeled 'Principal component 1/2' at cell_point_size = 0.5
- Bootstrapped branch matching is computed with Jaccard statistics and overlap coefficients, and detection rates lie in [0, 1] with low-rate branches flagged as uncertain (e.g. 0.615 branch)

## Limitations

- Branches with low bootstrap detection rates (e.g. the ~0.615 branch in HCA-BM) carry substantial topology-inference uncertainty and their differential topology p-values should be interpreted cautiously
- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or slingshot trajectories; interoperability with these methods is deferred to future work
- Simultaneously accounting for pseudotime inference uncertainty, gene expression/cell abundance uncertainty, and cross-sample/cross-cell variability remains an open research topic

## Evidence

- [discussion] In Lamian, TSCAN is used as the default method to construct pseudotime due to its flexibility and scalability.: "In Lamian, TSCAN is used as the default method to construct pseudotime due to its flexibility and scalability."
- [discussion] For each of the branch, we characterize its uncertainty using its detection rate in 1000 bootstrap samples.: "For each of the branch, we characterize its uncertainty using its detection rate in 1000 bootstrap samples."
- [methods] Here, we use example data hca_bm_saver, hca_bm_pca, hca_bm_cellanno to demonstrate their data structures.: "Here, we use example data **hca_bm_saver**, **hca_bm_pca**, **hca_bm_cellanno** to demonstrate their data structures."
- [methods] Lamian::plotmclust(res, cell_point_size = 0.5,: "Lamian::plotmclust(res, cell_point_size = 0.5,"
- [readme] Given scRNA-seq data from multiple biological samples with covariates (e.g. age, sex, sample type, disease status, etc.), this framework allows one to (1) construct cellular pseudotemporal trajectory, evaluate the uncertainty of the trajectory branching structure: "Given scRNA-seq data from multiple biological samples with covariates (e.g. age, sex, sample type, disease status, etc.), this framework allows one to (1) construct cellular pseudotemporal"
