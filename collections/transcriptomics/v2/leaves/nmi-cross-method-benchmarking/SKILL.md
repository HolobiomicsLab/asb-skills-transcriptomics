---
name: nmi-cross-method-benchmarking
description: Use when comparing community detection methods on personalized dynamic
  networks and you need a subject-consistency metric (NMI between donors) plus a non-parametric
  significance test (Wilcoxon rank-sum) per time point.
when_to_use_negative:
- Networks are not multi-subject (only a single network or a single time point) —
  static community-detection evaluation, not cross-subject NMI benchmarking, is appropriate
- Community labels are not discrete cluster assignments (e.g. continuous embeddings),
  where NMI between subjects is undefined without additional clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MuDCoD
  role: Primary method under evaluation; provides fit_predict community labels on
    the T x S network stack
  repo: https://github.com/bo1929/MuDCoD
- name: PisCES
  role: Baseline dynamic spectral clustering method (Liu et al. 2018) compared against
    via NMI
- name: Dozer
  role: Builds the personalized gene co-expression adjacency matrices whose community
    labels feed the NMI comparison
  repo: https://github.com/shanlu01/Dozer
- name: numpy
  role: Holds the multi-dimensional (subjects x time points) network array passed
    to MuDCoD fit_predict
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:5280b8cd2d380df60421b2ee66f2f737aff71645d1a20fc65b71bb5cd576232b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - Dozer
  - MuDCoD
  - PisCES
  - numpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# NMI cross-method benchmarking

## Summary

Evaluates whether a multi-subject dynamic community detection method (e.g. MuDCoD) produces more consistent community labels across subjects than baselines by computing pairwise subject-subject Normalized Mutual Information (NMI) per time point and testing differences with a Wilcoxon rank-sum test, as done for MuDCoD vs PisCES and static spectral clustering on the Jerber-2021 iPSC dopaminergic differentiation data.

## When to use

Use when comparing community detection methods on personalized dynamic networks and you need a subject-consistency metric (NMI between donors) plus a non-parametric significance test (Wilcoxon rank-sum) per time point.

## When NOT to use

- Networks are not multi-subject (only a single network or a single time point) — static community-detection evaluation, not cross-subject NMI benchmarking, is appropriate
- Community labels are not discrete cluster assignments (e.g. continuous embeddings), where NMI between subjects is undefined without additional clustering

## Inputs

- Per-method community label assignments (one label vector per subject per time point) from MuDCoD, PisCES, and static spectral clustering
- Stack of T x S personalized weighted adjacency matrices (n = top 3000 HVGs, top-5% edge threshold) from Dozer on the Jerber-2021 day-30/day-52 DA, Sert, and Epen1 cells

## Outputs

- Pairwise donor-donor NMI values per method per time point
- Donor-pair NMI histograms per method (Fig. 4a-b style)
- Wilcoxon rank-sum P-value comparing methods at day-52 (reported 0.0003)

## How to apply

Run each method (MuDCoD, PisCES, static spectral clustering) on the same stack of subject-by-time adjacency matrices to obtain community labels for every subject and time point. For each method and each time point, compute NMI between every pair of subjects' community labelings. Summarize the pairwise NMI distributions as histograms per method per time point (reproducing Fig. 4a-b style plots), then apply a Wilcoxon rank-sum test to the per-method NMI values at the target time point (day-52 in the Jerber-2021 reproduction, reported P = 0.0003 in favor of MuDCoD over PisCES). Use identical input networks and label counts (e.g. max_K = 50, n_iter = 30 for MuDCoD with alpha = 0.05*np.ones((T,2)), beta = 0.05*np.ones(S)) so the NMI comparison is apples-to-apples across methods.

## Related tools

- **MuDCoD** (Primary method under evaluation; provides fit_predict community labels on the T x S network stack) — https://github.com/bo1929/MuDCoD
- **PisCES** (Baseline dynamic spectral clustering method (Liu et al. 2018) compared against via NMI)
- **Dozer** (Builds the personalized gene co-expression adjacency matrices whose community labels feed the NMI comparison) — https://github.com/shanlu01/Dozer
- **numpy** (Holds the multi-dimensional (subjects x time points) network array passed to MuDCoD fit_predict)

## Examples

```
pred_MuDCoD = model.fit_predict(networks, alpha=0.05*np.ones((T,2)), beta=0.05*np.ones(S), max_K=50, n_iter=30, monitor_convergence=True)
```

## Evaluation signals

- MuDCoD's day-52 pairwise donor-donor NMI distribution is shifted above PisCES, with Wilcoxon rank-sum P ≈ 0.0003
- All methods are evaluated on the same T x S adjacency matrices (same subjects, time points, top-3000 HVG networks, top-5% edge threshold)
- NMI values lie in [0, 1] for every donor pair and every method/time point
- Per-donor static spectral clustering NMI should be no better than MuDCoD at day-52 if the dynamic smoothing across subjects is helping
- MuDCoD predictions can be sanity-checked against known labels by comparing pred_MuDCoD with true communities to evaluate accuracy (ARI)

## Limitations

- Wilcoxon P-value significance depends on the number of donor pairs (small donor counts yield limited power)
- The article reports no concrete numeric ARI values for simulation accuracy (code prints computed values but no outputs appear in the text), so method accuracy must be recomputed, not looked up
- NMI between donors measures label consistency, not biological correctness of modules; a method could be consistently wrong across donors

## Evidence

- [results] MuDCoD leads to significantly higher NMI scores between donors at day-52 compared to PisCES (Wilcoxon rank-sum test P-value of 0.0003).: [evidence span withheld — anchor status: not_found]
- [methods] Run MuDCoD iterative algorithm to find smoothed spectral representations: "Run MuDCoD iterative algorithm to find smoothed spectral representations of nodes, and then predict by clustering them to communities."
- [methods] You can compare `pred_MuDCoD` and `z_mus_dynamic_true` to evaluate the accuracy: "You can compare `pred_MuDCoD` and `z_mus_dynamic_true` to evaluate the accuracy"
- [readme] MuDCoD (Multi-subject Dynamic Community Detection) provides robust community detection in time-varying personalized networks modules.: "MuDCoD (Multi-subject Dynamic Community Detection) provides robust community detection in time-varying personalized networks modules."
- [abstract] MuDCoD effectively leverages apparent shared signals among networks of the subjects at individual time points: "MuDCoD effectively leverages apparent shared signals among networks of the subjects at individual time points, and performs robustly when there is no or little information sharing among the networks"
