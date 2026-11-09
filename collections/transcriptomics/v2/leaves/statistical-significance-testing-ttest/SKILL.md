---
name: statistical-significance-testing-ttest
description: Use when comparing a proposed method's averaged detection/classification
  metrics (e.g. AUROC at C^novel=1-4) against baseline methods across benchmark datasets
  and you need to confirm the improvement is significant, not noise.
when_to_use_negative:
- When you only have a single aggregated score per method with no per-dataset replicates
  — a t-test requires per-dataset metric vectors to test against
- When the goal is two-sided comparison of arbitrary methods rather than the one-tailed
  hypothesis that scNovel's novel rare cell detection advantage is real
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3307
tools:
- name: scNovel
  role: Proposed method whose averaged AUROC/AUPR/FPR95 performance vector is tested
    against baselines with the one-tailed t-test
  repo: github.com/chuanyang-Zheng/scNovel
- name: scLearn
  role: Second-best baseline; t-test quantifies scNovel's average 11.75% AUROC gain
    over it
- name: scPred
  role: Baseline method whose averaged metric vector enters the significance comparison
- name: scmap-cluster
  role: Baseline method whose averaged metric vector enters the significance comparison
- name: scmap-cell
  role: Baseline method whose averaged metric vector enters the significance comparison
- name: Python
  role: Language in which the averaged-metric table and one-tailed t-test are implemented
    (scNovel is an open-source Python package; SciPy/NumPy stack per README requirements)
  repo: github.com/chuanyang-Zheng/scNovel
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:4456ba413e26df31515c7530817637d6e3c8ba39ae26c0e96afd80430639952d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - scLearn
  - scNovel
  - scPred
  - scmap-cell
  - scmap-cluster
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# statistical-significance-testing-ttest

## Summary

Runs a one-tailed t-test over per-dataset averaged performance metrics (AUROC, AUPR, FPR95) to assess whether scNovel's novel rare cell detection advantage over scmap-cluster, scmap-cell, scPred and scLearn is statistically significant.

## When to use

Use when comparing a proposed method's averaged detection/classification metrics (e.g. AUROC at C^novel=1-4) against baseline methods across benchmark datasets and you need to confirm the improvement is significant, not noise.

## When NOT to use

- When you only have a single aggregated score per method with no per-dataset replicates — a t-test requires per-dataset metric vectors to test against
- When the goal is two-sided comparison of arbitrary methods rather than the one-tailed hypothesis that scNovel's novel rare cell detection advantage is real

## Inputs

- Per-dataset, per-C^novel metric arrays (AUROC, AUPR, FPR95) for scNovel, scmap-cluster, scmap-cell, scPred and scLearn on benchmark datasets (Zilionis, Darmanis, TM, PBMCbench)
- Method labels identifying which metric vector belongs to scNovel vs. each baseline

## Outputs

- Averaged performance table mirroring Appendix Table 2 (benchmark_performance_table.csv)
- One-tailed t-test p-value quantifying significance of scNovel's AUROC advantage over each baseline

## How to apply

First compute per-dataset metric values (primary metric: AUROC, with AUPR and FPR95 as secondary) for each method on each dataset/C^novel combination, treating novel-class cells as positives. Average the per-dataset metrics over all C^novel values (1, 2, 3, 4) for each method, mirroring Appendix Table 2. Then run a one-tailed t-test on the averaged per-dataset metric vectors, testing that scNovel's AUROC is greater than the baselines'. Validate significance against known ground truth: scNovel should be the only method with averaged AUROC above 94% (96.33% AUROC at C^novel=1) and should gain on average 11.75% AUROC over the second-best method scLearn (up to 16.26% in the paper). Save the assembled table as benchmark_performance_table.csv.

## Related tools

- **scNovel** (Proposed method whose averaged AUROC/AUPR/FPR95 performance vector is tested against baselines with the one-tailed t-test) — github.com/chuanyang-Zheng/scNovel
- **scLearn** (Second-best baseline; t-test quantifies scNovel's average 11.75% AUROC gain over it)
- **scPred** (Baseline method whose averaged metric vector enters the significance comparison)
- **scmap-cluster** (Baseline method whose averaged metric vector enters the significance comparison)
- **scmap-cell** (Baseline method whose averaged metric vector enters the significance comparison)
- **Python** (Language in which the averaged-metric table and one-tailed t-test are implemented (scNovel is an open-source Python package; SciPy/NumPy stack per README requirements)) — github.com/chuanyang-Zheng/scNovel

## Examples

```
from scipy import stats; stat, p = stats.ttest_ind(scNovel_aurocs, scLearn_aurocs, alternative='greater')
```

## Evaluation signals

- scNovel is the only method whose averaged AUROC exceeds 94% (96.33% AUROC at C^novel=1)
- scNovel gains on average 11.75% AUROC over the second-best method scLearn (up to 16.26% more AUROC reported)
- One-tailed t-test yields a significant p-value supporting the claim that scNovel's averaged performance advantage is not due to chance
- Assembled benchmark_performance_table.csv reproduces the structure of Appendix Table 2 with per-C^novel averages for all five methods
- FPR95 of scNovel is the lowest, AUROC and AUPR are the highest among compared methods after averaging

## Limitations

- Significance is assessed on averaged per-dataset metrics across C^novel = 1-4 on the specific benchmark suites (Zilionis, Darmanis, TM, PBMCbench); transfer to other datasets is not established by this test
- The paper acknowledges the score function behind the metrics could be improved, which bounds the meaningfulness of significance claims on absolute scores

## Evidence

- [other] Average the per-dataset metrics over all C^novel values and run a one-tailed t-test to assess the statistical significance of scNovel's advantage.: [evidence span withheld — anchor status: not_found]
- [abstract] scNovel significantly outperforms previous state-of-the-art novel cell detection models, reaching the most AUROC performance: "scNovel significantly outperforms previous state-of-the-art novel cell detection models, reaching the most AUROC performance(the only one method whose averaged AUROC results are above 94%, up to"
