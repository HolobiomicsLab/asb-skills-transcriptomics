---
name: single-cell-lineage-subsampling-simulation
description: Use when benchmarking differential trajectory-topology or cell-proportion
  tests on multi-sample scRNA-seq data by artificially removing cells from a known
  lineage and checking false positive rate at 0% removal and power as removal increases.
when_to_use_negative:
- The analysis goal is detecting differential gene expression or cell density along
  pseudotime rather than topology/cell-proportion changes driven by lineage composition
  shifts.
- The trajectory is built with Monocle2, Monocle3, or Slingshot — Lamian modules 1
  and 2 do not support these trajectory types (default is TSCAN).
- Only a single sample per condition is available, since the differential tests rely
  on cross-sample variability with multiple replicates.
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: Lamian
  role: Framework for inferring the pseudotime tree (infer_tree_structure), assessing
    branch uncertainty (evaluate_uncertainty), and testing differential topology with
    branchPropTest
  repo: https://github.com/Winnie09/Lamian
- name: TSCAN
  role: Default cluster-based minimum spanning tree method used by Lamian to construct
    the pseudotime trajectory
  repo: https://github.com/zji90/TSCAN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:03d398595affaedbe8fa8fa63409b349955a00604d26ee23cea4a61919f46149
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - TSCAN
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-lineage-subsampling-simulation

## Summary

Simulate lineage composition changes in multi-sample scRNA-seq data by subsampling a target lineage (e.g., myeloid cells) at graded removal percentages (0%–80%) across all or a subset of samples, then use Lamian's differential topology tests to benchmark type I error control and statistical power.

## When to use

Use when benchmarking differential trajectory-topology or cell-proportion tests on multi-sample scRNA-seq data by artificially removing cells from a known lineage and checking false positive rate at 0% removal and power as removal increases.

## When NOT to use

- The analysis goal is detecting differential gene expression or cell density along pseudotime rather than topology/cell-proportion changes driven by lineage composition shifts.
- The trajectory is built with Monocle2, Monocle3, or Slingshot — Lamian modules 1 and 2 do not support these trajectory types (default is TSCAN).
- Only a single sample per condition is available, since the differential tests rely on cross-sample variability with multiple replicates.

## Inputs

- Multi-sample scRNA-seq expression matrix (e.g., HCA-BM raw/SAVER counts from 8 healthy donors)
- Low-dimensional embedding (PCA, e.g., hca_bm_pca)
- Per-cell annotation with sample IDs and lineage labels (e.g., hca_bm_cellanno)
- Simulated design matrix specifying which samples belong to the removal group

## Outputs

- Per-branch detection rates at each removal level (from bootstrap tree reconstruction, n.permute >= 100)
- Branch cell proportions per sample per removal level
- Type I error estimate at 0% removal and power curve across removal levels for each simulation design
- Myeloid-branch detection rate at 80% removal (target ~0.106 in Sim 1)

## How to apply

Load multi-sample scRNA-seq data (HCA-BM: expression matrix/SAVER counts, PCA, cell annotation from 8 healthy donors) and identify the lineage to deplete (myeloid-lineage cells relative to the CD34+ origin). Infer the pseudotime tree with Lamian's infer_tree_structure (number.cluster = 5, origin.marker = 'CD34'). Subsample the lineage at removal levels 0%–80% under two designs: Simulation 1 removes myeloid cells from all 8 samples, Simulation 2 removes them from only 4 of 8 samples. For each level, run evaluate_uncertainty with n.permute >= 100 to estimate per-branch detection rates via bootstrap tree reconstruction using Jaccard statistics and overlap coefficient, and compute branch cell proportions per sample. Test differential topology with branchPropTest (binomial logistic regression) against a design matrix encoding the removal group assignment, then tabulate type I error at 0% and power across levels.

## Related tools

- **Lamian** (Framework for inferring the pseudotime tree (infer_tree_structure), assessing branch uncertainty (evaluate_uncertainty), and testing differential topology with branchPropTest) — https://github.com/Winnie09/Lamian
- **TSCAN** (Default cluster-based minimum spanning tree method used by Lamian to construct the pseudotime trajectory) — https://github.com/zji90/TSCAN

## Examples

```
res = infer_tree_structure(pca = hca_bm_pca, cellanno = hca_bm_cellanno, origin.marker = 'CD34', number.cluster = 5); result <- evaluate_uncertainty(res, n.permute = 100); res <- branchPropTest(res, design)
```

## Evaluation signals

- At 0% removal, differential topology test type I error is controlled near the nominal level (no significant myeloid-branch differences when there is no real difference).
- In Simulation 1, the myeloid-branch detection rate at 80% removal is approximately 0.106 (matching the reported reference value).
- Power in Simulation 2 increases monotonically as the percentage of removed cells increases from 0% to 80%.
- evaluate_uncertainty run with n.permute >= 100; branch uncertainty is quantified via detection rate using Jaccard statistics and overlap coefficient.
- Subsampling yields exactly 0–80% myeloid depletion in the designated 4 of 8 samples (Sim 2) or all 8 samples (Sim 1), with other lineages untouched.

## Limitations

- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or Slingshot trajectories; extension to such methods is future work.
- The differential topology test requires multiple biological samples per condition; the simulation depends on the 8-sample HCA-BM design (4 vs 4 for Sim 2).
- Pseudotime inference uncertainty is only partially accounted for; simultaneous modeling of all uncertainty sources is deferred to future research.

## Evidence

- [other] Lamian's differential topology tests controlled the probability of false positives (type I error rate) when there were no differences (0% of cells removed) and had increasing statistical power: [evidence span withheld — anchor status: not_found]
- [discussion] For each of the branch, we characterize its uncertainty using its detection rate in 1000 bootstrap samples.: "For each of the branch, we characterize its uncertainty using its detection rate in 1000 bootstrap samples."
- [discussion] To test differential topology, by default a binomial logistic regression model is fitted for each branch.: "To test differential topology, by default a binomial logistic regression model is fitted for each branch."
- [methods] res = infer_tree_structure(pca = man_tree_data[['pca']],: "res = infer_tree_structure(pca = man_tree_data[['pca']],"
- [methods] result <- evaluate_uncertainty(res, n.permute=5): "result <- evaluate_uncertainty(res, n.permute=5)"
- [discussion] In Lamian, TSCAN is used as the default method to construct pseudotime due to its flexibility and scalability.: "In Lamian, TSCAN is used as the default method to construct pseudotime due to its flexibility and scalability."
