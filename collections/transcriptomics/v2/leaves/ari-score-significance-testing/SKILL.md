---
name: ari-score-significance-testing
description: Use when you have paired per-dataset clustering accuracy scores (e.g.
  ARI) for a proposed method and one or more baselines over the same dataset set and
  need non-parametric significance testing.
when_to_use_negative:
- The ARI vectors are not paired (e.g. baselines were evaluated on different datasets),
  so the paired signed-rank assumption is violated
- Only a single clustering result per method is available with no per-dataset score
  breakdown, leaving nothing to pair across datasets
- The goal is clustering itself (e.g. running CASCC::run.CASCC or predictClusterK)
  rather than post-hoc benchmark significance analysis
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: stats::wilcox.test with paired=TRUE performs the paired Wilcoxon signed-rank
    tests on the ARI vectors
- name: CASCC_benchmark
  role: provides the per-dataset ARI benchmark results and code for the 15-dataset
    evaluation
  repo: https://github.com/LingyiC/CASCC_benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae283
    title: 'CASCC: a co-expression-assisted single-cell RNA-seq data clustering method'
  source_content_hash: sha256:c8e4b3144e41926f9de80d56b47924c996b27e08541794d8564de210d217a8c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LingyiC/CASCC_benchmark
  tools:
  - CASCC_benchmark
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# ari-score-significance-testing

## Summary

Paired Wilcoxon signed-rank testing of per-dataset ARI scores to establish whether a clustering method's improvement over baselines is statistically significant, as done for CASCC versus six single-cell clustering methods across the 15-dataset benchmark.

## When to use

Use when you have paired per-dataset clustering accuracy scores (e.g. ARI) for a proposed method and one or more baselines over the same dataset set and need non-parametric significance testing.

## When NOT to use

- The ARI vectors are not paired (e.g. baselines were evaluated on different datasets), so the paired signed-rank assumption is violated
- Only a single clustering result per method is available with no per-dataset score breakdown, leaving nothing to pair across datasets
- The goal is clustering itself (e.g. running CASCC::run.CASCC or predictClusterK) rather than post-hoc benchmark significance analysis

## Inputs

- Per-dataset ARI score matrix (15 benchmark datasets) with columns for CASCC and the six baselines CIDR, RaceID3, SC3, Seurat V4, SIMLR, TSCAN
- Benchmark results from the LingyiC/CASCC_benchmark repository

## Outputs

- supp_table_S4_wilcoxon_significance.csv with one row per CASCC-vs-baseline comparison, each containing a Wilcoxon test statistic and p-value (6 rows total)

## How to apply

Load the per-dataset ARI score matrix containing CASCC and the six baselines (CIDR, RaceID3, SC3, Seurat V4, SIMLR, TSCAN) over the 15 Table S1 benchmark datasets from the LingyiC/CASCC_benchmark results. For each baseline, run a paired Wilcoxon signed-rank test in R with stats::wilcox.test(paired=TRUE), comparing the CASCC ARI vector against that baseline's ARI vector over the same 15 datasets; pairing is required because the same datasets are shared across methods. Collect each test's statistic and p-value into a one-row-per-comparison table and write it to supp_table_S4_wilcoxon_significance.csv. The paper reports that CASCC's ARI differences versus all six compared methods are statistically significant (Supplementary Table S4), so the completed table should show significance for every comparison.

## Related tools

- **R** (stats::wilcox.test with paired=TRUE performs the paired Wilcoxon signed-rank tests on the ARI vectors)
- **CASCC_benchmark** (provides the per-dataset ARI benchmark results and code for the 15-dataset evaluation) — https://github.com/LingyiC/CASCC_benchmark

## Examples

```
wilcox.test(ari$CASCC, ari$SIMLR, paired=TRUE)
```

## Evaluation signals

- supp_table_S4_wilcoxon_significance.csv contains exactly 6 rows, one per CASCC-vs-baseline comparison (CIDR, RaceID3, SC3, Seurat V4, SIMLR, TSCAN)
- Every row contains both a non-null test statistic and a non-null p-value; any row missing either fails the check
- The table's significance pattern reproduces the results of the paper's Supplementary Table S4 (CASCC's ARI differences significant versus all six methods)
- Each test was run paired=TRUE on the same 15 benchmark datasets shared by the CASCC and baseline ARI vectors
- ARI score matrix rows correspond to the Table S1 benchmark datasets with no missing scores for either method in a comparison

## Limitations

- Wilcoxon signed-rank tests are non-parametric and only assess rank-based differences in paired ARI scores; they do not convey effect size on the ARI scale
- Statistical conclusions are tied to the 15-dataset benchmark; adding or removing evaluation datasets changes the paired vectors and the test outcomes
- The skill validates significance of differences, not the magnitude of ARI improvement reported in Supplementary Table S4

## Evidence

- [supplementary] Wilcoxon signed-rank tests confirm that CASCC's ARI performance differences versus all other compared methods are statistically significant: [evidence span withheld — anchor status: not_found]
- [methods] Wilcoxon test workflow in R: [evidence span withheld — anchor status: not_found]
- [readme] Benchmark repository pointer: "https://github.com/LingyiC/CASCC_benchmark"
- [readme] Published method reference: "The users can refer to our published manuscript, available at [this link](https://doi.org/10.1093/bioinformatics/btae283), for a detailed description of the method."
