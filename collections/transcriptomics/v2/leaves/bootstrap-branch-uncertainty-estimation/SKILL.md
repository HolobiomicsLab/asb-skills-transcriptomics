---
name: bootstrap-branch-uncertainty-estimation
description: Use when a multi-sample scRNA-seq trajectory (e.g. TSCAN cluster-based
  MST on HCA-BM data) contains branches whose stability is uncertain, and you need
  per-branch detection rates or sample-level branch-proportion variability before
  testing differential topology.
when_to_use_negative:
- Lamian modules 1 and 2, including this branch-uncertainty quantification, do not
  support Monocle2, Monocle3, or Slingshot trajectories — do not apply to trees built
  by those tools.
- The goal is differential gene expression or cell-density testing along a fixed pseudotime,
  not characterization of trajectory branching structure itself.
- Data are single-sample per condition without sample-to-sample replication, since
  branch proportion variability is interpreted across biological samples.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Lamian
  role: Provides evaluate_uncertainty, which bootstraps cells, rebuilds the TSCAN-based
    tree, and computes branch detection rates via Jaccard statistics and overlap coefficients
  repo: https://github.com/Winnie09/Lamian
- name: TSCAN
  role: Default method used to construct the cluster-based minimum spanning tree whose
    branches are evaluated for uncertainty
  repo: https://github.com/zji90/TSCAN
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:a51ec5d8c903d05ffdec9f853b075115262caff7a261908fe5be166361580f54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - TSCAN
  - branchPropTest
  - evaluate_uncertainty
  - plotmclust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# bootstrap-branch-uncertainty-estimation

## Summary

Quantify the uncertainty of each branch in a cluster-based minimum spanning tree pseudotime trajectory by rebuilding the tree on bootstrap-resampled cells and computing each branch's detection rate, i.e. the percentage of bootstrap settings in which the branch is matched. This is Lamian's approach to characterizing pseudotime trajectory/branching instability before any differential topology test.

## When to use

Use when a multi-sample scRNA-seq trajectory (e.g. TSCAN cluster-based MST on HCA-BM data) contains branches whose stability is uncertain, and you need per-branch detection rates or sample-level branch-proportion variability before testing differential topology.

## When NOT to use

- Lamian modules 1 and 2, including this branch-uncertainty quantification, do not support Monocle2, Monocle3, or Slingshot trajectories — do not apply to trees built by those tools.
- The goal is differential gene expression or cell-density testing along a fixed pseudotime, not characterization of trajectory branching structure itself.
- Data are single-sample per condition without sample-to-sample replication, since branch proportion variability is interpreted across biological samples.

## Inputs

- Output of Lamian::infer_tree_structure on a gene-by-cell expression matrix (e.g. hca_bm_saver) plus PCA embedding (hca_bm_pca) and cell-to-sample annotation (hca_bm_cellanno)
- Cluster-based MST trajectory with a defined origin marker (e.g. origin.marker = 'CD34', number.cluster = 5)
- n.permute parameter setting the number of bootstrap resampling iterations

## Outputs

- Per-branch detection rate: fraction of bootstrap settings in which the branch was matched (e.g. ~0.938, 0.953, 0.615 for the HCA-BM myeloid, erythroid, lymphoid branches)
- Per-branch sample-level cell proportion means and standard deviations (e.g. ~41.1%, 48.4%, 10.5% for HCA-BM branches)
- Branch-matching statistics (Jaccard statistics and overlap coefficients) between bootstrapped and original trees

## How to apply

First infer the trajectory with infer_tree_structure (e.g. on the HCA-BM PCA embedding with origin.marker = 'CD34' and number.cluster = 5, yielding the three-branch tree 2 --> 3 --> 1, 2 --> 4, 2 --> 5). Then call evaluate_uncertainty on the inferred tree with n.permute >= 100 (up to 10,000 bootstraps for final reporting). In each bootstrap, resample cells, reconstruct the tree, and match bootstrapped branches to the original branches using Jaccard statistics and overlap coefficients; a branch's detection rate is the percentage of bootstrap settings with a match. The function also returns per-branch cell proportion means and standard deviations across samples. Interpret low detection rates (e.g. ~0.615 for the smallest HCA-BM lymphoid branch, or ~0.106 for the myeloid branch after 80% cell removal) as high branch uncertainty, which informs interpretation of downstream branchPropTest differential topology results.

## Related tools

- **Lamian** (Provides evaluate_uncertainty, which bootstraps cells, rebuilds the TSCAN-based tree, and computes branch detection rates via Jaccard statistics and overlap coefficients) — https://github.com/Winnie09/Lamian
- **TSCAN** (Default method used to construct the cluster-based minimum spanning tree whose branches are evaluated for uncertainty) — https://github.com/zji90/TSCAN

## Examples

```
result <- evaluate_uncertainty(res, n.permute=100)
```

## Evaluation signals

- On the 8-donor HCA-BM dataset with number.cluster = 5 and origin.marker = 'CD34', branch detection rates should be approximately 0.938, 0.953, and 0.615, with branch cell proportions around 41.1%, 48.4%, and 10.5%.
- In the myeloid-removal Simulation 1 at 80% removal, the myeloid-branch detection rate should drop to approximately 0.106.
- Each reported detection rate must correspond to the percentage of bootstrap settings with a matched branch, matched via Jaccard statistics and overlap coefficients.
- Per-branch sample proportion means and standard deviations must be returned alongside detection rates for downstream branchPropTest differential topology testing.
- Bootstrapped trees must use the same cell-to-sample structure as the original data; detection rates should be based on n.permute >= 100 bootstrap iterations for stable estimates (the article's real-data analysis used 1000 bootstrap samples).

## Limitations

- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or slingshot trajectories; adapting to those trajectory inference methods is deferred to future work.
- Simultaneously accounting for pseudotime inference uncertainty, gene expression and cell abundance uncertainty, and cross-sample/cross-cell variability remains a future research topic.
- Branch detection rates are limited to the branches of the originally inferred tree; highly unstable branches (low detection rates) may indicate the inferred topology itself is unreliable rather than biologically variable.

## Evidence

- [discussion] For each of the branch, we characterize its uncertainty using its detection rate in 1000 bootstrap samples.: "For each of the branch, we characterize its uncertainty using its detection rate in 1000 bootstrap samples."
- [methods] Quantify branch uncertainty with evaluate_uncertainty, matching bootstrapped branches to original branches using Jaccard statistics and overlap coefficients over 10,000 bootstraps: [evidence span withheld — anchor status: not_found]
- [methods] result <- evaluate_uncertainty(res, n.permute=5): "result <- evaluate_uncertainty(res, n.permute=5)"
- [results] confirm the branch detection rates (~0.938, 0.953, 0.615), branch cell proportions (~41.1%, 48.4%, 10.5%): [evidence span withheld — anchor status: not_found]
