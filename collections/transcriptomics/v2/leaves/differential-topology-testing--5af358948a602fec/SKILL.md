---
name: differential-topology-testing
description: Use when you have pseudotime trajectories inferred from multiple samples
  (e.g., 8 HCA-BM donors) and want to know whether covariates change which trajectory
  branches exist, with uncertainty-aware, sample-level statistical inference.
when_to_use_negative:
- Your trajectory inference does not produce a cluster-based tree compatible with
  Lamian's modules — the article notes Lamian modules 1 and 2 do not support Monocle2,
  Monocle3, or Slingshot trajectories
- You need to compare exactly one sample per condition — the article states condiments
  assumes each condition has one sample and Lamian instead requires sample-level variation
  to be estimable across replicates
- Your question is about gene expression along pseudotime or cell density rather than
  the branching structure itself (use Lamian's TDE/TCD modules instead)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Lamian
  role: Provides infer_tree_structure to build the pseudotime tree, evaluate_uncertainty
    for branch detection-rate uncertainty, and branchPropTest (binomial logistic regression)
    for the differential topology test
  repo: https://github.com/Winnie09/Lamian
- name: TSCAN
  role: Default cluster-based minimum spanning tree method Lamian uses internally
    to construct the pseudotime trajectory whose branches are tested
  repo: https://github.com/zji90/TSCAN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:03f5917c488e637f344599a194f0dbf12b65497215c71ee00fdd3c4bd2e70253
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - TSCAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# differential-topology-testing

## Summary

Test whether the branching structure (topology) of a pseudotime trajectory differs across sample covariates in multi-sample scRNA-seq data, using Lamian's branch detection-rate uncertainty estimates and branchPropTest binomial logistic regression. The skill was validated on the HCA bone marrow dataset by subsampling myeloid cells and showing controlled type I error with monotone power gains.

## When to use

Use when you have pseudotime trajectories inferred from multiple samples (e.g., 8 HCA-BM donors) and want to know whether covariates change which trajectory branches exist, with uncertainty-aware, sample-level statistical inference.

## When NOT to use

- Your trajectory inference does not produce a cluster-based tree compatible with Lamian's modules — the article notes Lamian modules 1 and 2 do not support Monocle2, Monocle3, or Slingshot trajectories
- You need to compare exactly one sample per condition — the article states condiments assumes each condition has one sample and Lamian instead requires sample-level variation to be estimable across replicates
- Your question is about gene expression along pseudotime or cell density rather than the branching structure itself (use Lamian's TDE/TCD modules instead)

## Inputs

- Expression matrix (e.g., hca_bm_saver SAVER-smoothed counts)
- Low-dimensional embedding / PCA (e.g., hca_bm_pca)
- Cell annotation with sample IDs and cell type labels (e.g., hca_bm_cellanno, myeloid lineage relative to CD34+ origin)
- Design matrix or sample covariate table encoding the condition to test
- Inferred pseudotime tree (infer_tree_structure output)

## Outputs

- Per-branch detection rates from bootstrap tree reconstruction (uncertainty estimates)
- Differential topology test p-values per branch (binomial logistic regression on branch cell proportions)
- Type I error rate (at null/0% removal) and power/detection-rate curves across removal levels (e.g., myeloid-branch rate ~0.106 at 80% removal in Sim 1)

## How to apply

Infer the pseudotime tree with Lamian's infer_tree_structure (e.g., number.cluster = 5, origin.marker = 'CD34' on the HCA-BM data). Estimate each branch's uncertainty via evaluate_uncertainty (n.permute >= 100), which reconstructs the tree in bootstrap samples using Jaccard statistics and overlap coefficients, yielding a per-branch detection rate. Compute branch cell proportions per sample and fit a binomial logistic regression (branchPropTest, optionally multinomial) against the design matrix encoding the simulated or real covariate design (e.g., 4 of 8 samples with myeloid cells removed at 0%-80%). Judge success by type I error near the nominal level when no branches differ (0% removal) and monotonically increasing detection rate/power as the removal proportion grows; in Simulation 1 the myeloid-branch detection rate at 80% removal should approximate the reference value ~0.106.

## Related tools

- **Lamian** (Provides infer_tree_structure to build the pseudotime tree, evaluate_uncertainty for branch detection-rate uncertainty, and branchPropTest (binomial logistic regression) for the differential topology test) — https://github.com/Winnie09/Lamian
- **TSCAN** (Default cluster-based minimum spanning tree method Lamian uses internally to construct the pseudotime trajectory whose branches are tested) — https://github.com/zji90/TSCAN

## Examples

```
res = infer_tree_structure(pca = man_tree_data[['pca']]); result <- evaluate_uncertainty(res, n.permute=100)
```

## Evaluation signals

- Type I error rate is controlled (near the nominal significance level) when 0% of cells are removed, i.e., no real topological difference exists
- Statistical power / detection rate increases monotonically with the percentage of myeloid cells removed (0%-80%) in Simulation 2
- Myeloid-branch detection rate at 80% removal in Simulation 1 approximates the reference value ~0.106
- evaluate_uncertainty run with n.permute >= 100 produces a finite detection rate for every branch via Jaccard statistics and overlap coefficient
- Branch cell proportions per sample are computable for all 8 HCA-BM samples and feed a successfully converged binomial logistic regression in branchPropTest

## Limitations

- Branch uncertainty is quantified by detection rate in bootstrap samples; simultaneously accounting for pseudotime inference uncertainty, gene expression/cell abundance uncertainty, and cross-sample/cross-cell variability remains a future research topic
- Simulation evidence for topology is one part of the validation; simulations for TDE, TCD, and XCD detection are deferred to Supplementary Notes
- Visualization/plotting functions require ComplexHeatmap; if installation fails and the import is removed, heatmap plots will not work although tests still run

## Evidence

- [discussion] To test differential topology, by default a binomial logistic regression model is fitted for each branch.: "To test differential topology, by default a binomial logistic regression model is fitted for each branch."
- [discussion] For each of the branch, we characterize its uncertainty using its detection rate in 1000 bootstrap samples.: "For each of the branch, we characterize its uncertainty using its detection rate in 1000 bootstrap samples."
- [methods] result <- evaluate_uncertainty(res, n.permute=5): "result <- evaluate_uncertainty(res, n.permute=5)"
- [abstract] Here, we introduce Lamian, a comprehensive and statistically-rigorous computational framework for differential multi-sample pseudotime analysis.: "Here, we introduce Lamian, a comprehensive and statistically-rigorous computational framework for differential multi-sample pseudotime analysis."
- [readme] construct cellular pseudotemporal trajectory, evaluate the uncertainty of the trajectory branching structure: "construct cellular pseudotemporal trajectory, evaluate the uncertainty of the trajectory branching structure"
