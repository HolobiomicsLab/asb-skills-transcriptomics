---
name: rank-sum-statistical-testing
description: Use when two independent sets of scores must be compared for a significant
  shift — e.g. donor-pair NMI values from MuDCoD vs. PisCES at one time point — and
  a nonparametric test on unpaired samples is appropriate.
when_to_use_negative:
- When the score samples are paired (same donor pair scored under both methods with
  matching rank structure) — a paired Wilcoxon signed-rank test would be the correct
  variant, not the unpaired rank-sum test used here.
- When the outcome is a continuous measurement across many observations rather than
  a small set of similarity scores per donor pair, where a parametric or permutational
  test may offer more power.
- When no baseline comparison exists (e.g. evaluating a single method's accuracy against
  ground truth) — ARI against true labels is the article's suggested alternative.
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MuDCoD
  role: Method under test; its fit_predict community labels generate the NMI score
    distribution compared with the rank-sum test
  repo: https://github.com/bo1929/MuDCoD
- name: PisCES
  role: Baseline dynamic community-detection method whose day-52 NMI scores form the
    comparison sample in the rank-sum test
- name: numpy
  role: Holds the T x S adjacency stack and NMI score arrays passed to the statistical
    test
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:171440f8ab64b0b2dec51fc039ee5d01c520441f958e0405f34eaf4a2a262917
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - Dozer
  - MuDCoD
  - PisCES
  - numpy
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# rank-sum-statistical-testing

## Summary

Applies the Wilcoxon rank-sum (Mann–Whitney) nonparametric test to compare distributions of donor-pair NMI scores between two community-detection methods (MuDCoD vs. PisCES) at a fixed time point, e.g. day-52 of the Jerber-2021 dopaminergic differentiation dataset, yielding a significance p-value for method superiority.

## When to use

Use when two independent sets of scores must be compared for a significant shift — e.g. donor-pair NMI values from MuDCoD vs. PisCES at one time point — and a nonparametric test on unpaired samples is appropriate.

## When NOT to use

- When the score samples are paired (same donor pair scored under both methods with matching rank structure) — a paired Wilcoxon signed-rank test would be the correct variant, not the unpaired rank-sum test used here.
- When the outcome is a continuous measurement across many observations rather than a small set of similarity scores per donor pair, where a parametric or permutational test may offer more power.
- When no baseline comparison exists (e.g. evaluating a single method's accuracy against ground truth) — ARI against true labels is the article's suggested alternative.

## Inputs

- Per-method NMI score vectors at a fixed time point (one NMI value per donor pair; e.g. day-52 MuDCoD vs. PisCES)
- Predicted community label matrices (pred_MuDCoD and baseline partitions) from which NMI values are computed

## Outputs

- Wilcoxon rank-sum test statistic
- Two-sided or one-sided p-value (reproduced value: 0.0003 at day-52)
- Significance decision (MuDCoD > PisCES donor-donor module similarity at day-52)

## How to apply

First run the community-detection methods (MuDCoD with alpha=0.05*np.ones((T,2)), beta=0.05*np.ones(S), max_K=50, n_iter=30; plus PisCES and per-donor static spectral clustering baselines) on the T x S stack of donor adjacency matrices built with Dozer from the top-3000 highly variable genes. For each method and time point, compute all pairwise donor-donor NMI scores, giving one distribution of NMI values per method per time point. At the target time point (day-52), apply the Wilcoxon rank-sum test to the two independent samples of NMI values (one per method), testing the alternative that MuDCoD's NMI distribution is shifted higher; the Jerber-2021 reproduction reports P = 0.0003. Rank-based testing is chosen because NMI distributions across donor pairs are non-normal and sample sizes equal the number of donor pairs, so normal-theory t-tests are not justified.

## Related tools

- **MuDCoD** (Method under test; its fit_predict community labels generate the NMI score distribution compared with the rank-sum test) — https://github.com/bo1929/MuDCoD
- **PisCES** (Baseline dynamic community-detection method whose day-52 NMI scores form the comparison sample in the rank-sum test)
- **numpy** (Holds the T x S adjacency stack and NMI score arrays passed to the statistical test)

## Examples

```
from scipy.stats import ranksums; stat, p = ranksums(nmi_mudcod_day52, nmi_pisces_day52, alternative='greater')
```

## Evaluation signals

- The Wilcoxon rank-sum p-value on day-52 MuDCoD-vs-PisCES NMI values reproduces the reported P = 0.0003.
- Each input sample contains one NMI value per donor pair, and all NMI values lie in [0, 1].
- The test is applied to independent (unpaired) samples — donor pairs are pooled across methods, matching the rank-sum (not signed-rank) design.
- The same test applied at day-30 should not necessarily show the day-52-level significance, consistent with the paper's day-30 vs. day-52 contrast.

## Limitations

- The number of rank-sum observations equals the number of donor pairs (small n), limiting power and making results sensitive to ties among NMI values.
- A significant rank-sum p-value at day-52 does not control for multiplicity across other time points or additional baseline methods.
- No concrete numeric test statistics appear in the paper's text (the article notes missing numeric outputs), so exact reproduction depends on the NMI computation details rather than on published numbers.

## Evidence

- [other] MuDCoD leads to significantly higher NMI scores between donors at day-52 compared to PisCES (Wilcoxon rank-sum test P-value of 0.0003).: [evidence span withheld — anchor status: not_found]
- [abstract] MuDCoD effectively leverages apparent shared signals among networks of the subjects at individual time points, and performs robustly when there is no or little information sharing among the networks: "MuDCoD effectively leverages apparent shared signals among networks of the subjects at individual time points, and performs robustly when there is no or little information sharing among the networks"
- [methods] You can compare `pred_MuDCoD` and `z_mus_dynamic_true` to evaluate the accuracy: "You can compare `pred_MuDCoD` and `z_mus_dynamic_true` to evaluate the accuracy"
