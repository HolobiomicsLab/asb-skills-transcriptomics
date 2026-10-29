---
name: differential-topology-testing-sc-rna-seq
description: Use when you have multi-sample scRNA-seq (e.g., the 8-donor HCA bone
  marrow dataset with a sex covariate) and need to test whether trajectory tree topology
  or branch cell proportions change between conditions.
when_to_use_negative:
- Your data are Monocle2, Monocle3, or Slingshot trajectories — Lamian modules 1 and
  2 do not support these and tailoring them is deferred to future work.
- You only need gene expression or cell-density changes along pseudotime rather than
  branching-structure/topology or branch-proportion differences.
- Each condition has a single sample with no biological replicates, so cross-sample
  variability cannot be modeled.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Lamian
  role: Provides infer_tree_structure, evaluate_uncertainty, plotmclust, and branchPropTest
    for topology quantification and differential topology testing
  repo: https://github.com/Winnie09/Lamian
- name: TSCAN
  role: Cluster-based minimum spanning tree method used internally as the default
    pseudotime/trajectory construction method
  repo: https://github.com/zji90/TSCAN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:da6c9ee0add8063ea1f3e3143f27d3234e805e9dc290d4de7f4ae22760b5245e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - TSCAN
  - branchPropTest
  - plotmclust
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# differential-topology-testing-sc-rna-seq

## Summary

Quantify and test whether pseudotemporal trajectory branching structure (topology) differs across sample covariates in multi-sample scRNA-seq data using Lamian: bootstrapped branch detection rates plus binomial logistic regression of branch cell proportions.

## When to use

Use when you have multi-sample scRNA-seq (e.g., the 8-donor HCA bone marrow dataset with a sex covariate) and need to test whether trajectory tree topology or branch cell proportions change between conditions.

## When NOT to use

- Your data are Monocle2, Monocle3, or Slingshot trajectories — Lamian modules 1 and 2 do not support these and tailoring them is deferred to future work.
- You only need gene expression or cell-density changes along pseudotime rather than branching-structure/topology or branch-proportion differences.
- Each condition has a single sample with no biological replicates, so cross-sample variability cannot be modeled.

## Inputs

- Multi-sample scRNA-seq expression matrix (e.g., gene-by-cell counts from HCA bone marrow, 8 healthy donors, 10x Genomics)
- Low-dimensional cell embedding (PCA; hca_bm_pca example data)
- Cell-to-sample annotation with covariates (hca_bm_cellanno; e.g., sex design vector c(0, rep(1,4), rep(0,3)))
- Optional cell-origin markers (e.g., CD34) to root the tree

## Outputs

- Inferred tree structure (branch definitions, e.g., 2 --> 3 --> 1, 2 --> 4, 2 --> 5)
- Per-branch detection rates (fraction of bootstrap settings with a branch match), means, and standard deviations
- Per-branch binomial logistic regression p-values for topology change across covariates
- Tree visualization (plotmclust PCA plot with cell_point_size = 0.5, e.g., saved as tree_structure.png)

## How to apply

First infer the cluster-based minimum spanning tree trajectory with Lamian's infer_tree_structure (TSCAN is the default tree builder; e.g., origin.marker = 'CD34' and number.cluster = 5 on the HCA bone marrow data yields a three-branch tree 2-->3-->1, 2-->4, 2-->5). Then call evaluate_uncertainty to match bootstrapped branches to original branches via Jaccard statistics and overlap coefficients over the bootstrap replicates (the article reports 1000 bootstrap samples as the detection-rate basis), yielding per-branch detection rates, proportion means, and standard deviations. Finally, run branchPropTest, which by default fits a binomial logistic regression of each branch's cell proportion on the sample covariate (e.g., sex = c(0, rep(1,4), rep(0,3))), returning per-branch p-values for differential topology. In the HCA-BM validation, detection rates of ~0.938/0.953/0.615, branch proportions ~41.1%/48.4%/10.5%, and sex p-values of 0.35/0.64/0.94 reproduce the published result of no significant topology change between male and female donors.

## Related tools

- **Lamian** (Provides infer_tree_structure, evaluate_uncertainty, plotmclust, and branchPropTest for topology quantification and differential topology testing) — https://github.com/Winnie09/Lamian
- **TSCAN** (Cluster-based minimum spanning tree method used internally as the default pseudotime/trajectory construction method) — https://github.com/zji90/TSCAN

## Examples

```
res = infer_tree_structure(pca = man_tree_data[['pca']], origin.marker='CD34', number.cluster=5); result <- evaluate_uncertainty(res, n.permute=5); plotmclust(res, cell_point_size=0.5)
```

## Evaluation signals

- HCA-BM branch detection rates reproduce ~0.938, 0.953, and 0.615 across the three branches
- Branch cell proportions reproduce ~41.1%, 48.4%, and 10.5%
- Sex covariate p-values from branchPropTest reproduce ~0.35 (myeloid), 0.64 (erythroid), and 0.94 (lymphoid), i.e., no significant topology change
- infer_tree_structure with origin.marker='CD34' and number.cluster=5 yields the expected three-branch tree 2 --> 3 --> 1, 2 --> 4, 2 --> 5
- plotmclust output renders cells on Principal component 1/2 axes with the specified cell_point_size

## Limitations

- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or Slingshot trajectories; interoperability is a stated future direction.
- Trajectory/pseudotime inference uncertainty, cell abundance uncertainty, and cross-sample variability cannot all be accounted for simultaneously; this remains an open research topic.
- A systematic comparison of cell-harmonization methods is explicitly beyond the scope of the study.

## Evidence

- [discussion] To test differential topology, by default a binomial logistic regression model is fitted for each branch.: "To test differential topology, by default a binomial logistic regression model is fitted for each branch."
- [discussion] Branch detection rate via bootstrapping: "For each of the branch, we characterize its uncertainty using its detection rate in 1000 bootstrap samples."
- [discussion] TSCAN as default tree builder: "In Lamian, TSCAN is used as the default method to construct pseudotime due to its flexibility and scalability."
- [methods] infer_tree_structure call: "res = infer_tree_structure(pca = man_tree_data[['pca']],"
- [readme] Installation from GitHub: [evidence span withheld — anchor status: external_doc]
