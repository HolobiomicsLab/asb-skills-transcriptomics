---
name: wilcoxon-rank-sum-significance-comparison
description: Use when comparing two methods' distributions of per-run statistical
  significance (e.g. DA-test P-values over 100 simulation replicates) and you need
  a non-parametric rank-based test of whether one method attains systematically smaller
  P-values.
when_to_use_negative:
- When the per-run significance values are paired across runs rather than independent
  samples — a rank-sum test on independent samples would be inappropriate
- When only single point-estimate statistics per method exist (no replicate-run distributions),
  since the rank-sum test requires distributions of values to compare
- When the goal is a parametric comparison of means with verified normality; a t-test
  would have more power in that setting
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: ELVAR
  role: Provides the cell-attribute aware EVA clustering + DoDA negative-binomial
    DA-testing pipeline whose per-run P-value distribution is the test group
  repo: https://github.com/aet21/ELVAR
- name: R
  role: Implementation language; wilcox.test performs the one-sided Wilcoxon rank-sum
    comparison of P-value distributions
  repo: https://github.com/aet21/ELVAR
- name: dplyr
  role: Used in the ELVAR methods pipeline to manipulate per-run DA statistics before
    testing
  repo: https://github.com/aet21/ELVAR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:e71bcf65e11aa14ed0164a756882c5e1d6b7d32ce8d75c94d081937f2ff58e7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/ELVAR
  tools:
  - ELVAR
  - EVA
  - Matrix
  - R
  - Seurat
  - dplyr
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# wilcoxon-rank-sum-significance-comparison

## Summary

A benchmarking skill from the ELVAR paper: apply one-sided Wilcoxon rank-sum tests to compare distributions of negative-binomial-regression DA-test P-values produced over 100 simulation runs, establishing whether cell-attribute aware EVA clustering (ELVAR) yields significantly stronger differential-abundance significance than Louvain-based baselines (LVdet, LVnonseq).

## When to use

Use when comparing two methods' distributions of per-run statistical significance (e.g. DA-test P-values over 100 simulation replicates) and you need a non-parametric rank-based test of whether one method attains systematically smaller P-values.

## When NOT to use

- When the per-run significance values are paired across runs rather than independent samples — a rank-sum test on independent samples would be inappropriate
- When only single point-estimate statistics per method exist (no replicate-run distributions), since the rank-sum test requires distributions of values to compare
- When the goal is a parametric comparison of means with verified normality; a t-test would have more power in that setting

## Inputs

- Vector of 100 negative-binomial-regression DA-test P-values from the ELVAR/EVA pipeline runs
- Vector of 100 DA-test P-values from the LVdet (sequential deterministic Louvain) baseline runs
- Vector of 100 DA-test P-values from the LVnonseq (non-sequential stochastic Louvain) baseline runs
- Simulated multi-mouse scRNA-Seq dataset (993 cells, perturbation frequency 0.25 young vs 0.5 old) used to generate all per-run statistics

## Outputs

- Two one-sided Wilcoxon rank-sum P-values (ELVAR vs LVdet, ELVAR vs LVnonseq)
- Per-method DA statistics table saved as fig2e_wilcoxon_pvalues.csv

## How to apply

Run each clustering-DA pipeline (ELVAR with Eva_partitions at resolution=1, threshold=1e-7, Vattr.name='age', alpha optimized over 0.1-0.9 via SelOptAlpha; and the Louvain baselines LVdet and LVnonseq) on identical simulated datasets for 100 runs, each time recording negative-binomial-regression DA-test P-values from DoDA (varDA = cell-state label, varREP = mouse replicate). Collect the 100 P-values per method into per-method vectors. Apply a one-sided Wilcoxon rank-sum test (alternative = ELVAR smaller) comparing the ELVAR P-value distribution against each baseline's distribution. As in Fig. 2e of the paper, success is when ELVAR shows significantly stronger (smaller) DA-test P-values than both LVdet and LVnonseq; save the per-method DA statistics and the two Wilcoxon rank-sum P-values (e.g. to fig2e_wilcoxon_pvalues.csv).

## Related tools

- **ELVAR** (Provides the cell-attribute aware EVA clustering + DoDA negative-binomial DA-testing pipeline whose per-run P-value distribution is the test group) — https://github.com/aet21/ELVAR
- **R** (Implementation language; wilcox.test performs the one-sided Wilcoxon rank-sum comparison of P-value distributions) — https://github.com/aet21/ELVAR
- **dplyr** (Used in the ELVAR methods pipeline to manipulate per-run DA statistics before testing) — https://github.com/aet21/ELVAR

## Examples

```
wilcox.test(elvar_pv, lvdet_pv, alternative = "less")
```

## Evaluation signals

- Both Wilcoxon rank-sum P-values (ELVAR vs LVdet and ELVAR vs LVnonseq) should be significant, confirming ELVAR's DA-test P-values are stochastically smaller, reproducing Fig. 2e
- Each method's input vector must contain exactly 100 DA-test P-values, one per simulation run
- All methods must be run on the identical simulated 993-cell multi-mouse dataset (perturbation frequency 0.25 young to 0.5 old) so the comparison is like-for-like
- Per-method DA statistics and the two rank-sum P-values are written to fig2e_wilcoxon_pvalues.csv with expected columns/values
- DoDA calls must use varDA = cell-state label and varREP = mouse replicate so replicate structure is respected in the NBR regression

## Limitations

- ELVAR's improvement over its Louvain-analog was more pronounced in studies with weaker clustering structure and less noticeable in studies with stronger clustering
- ELVAR incurs increased computational complexity and runtimes compared to Milo or the Louvain-analog (deemed not practically prohibitive)
- Identifying an optimal alpha parameter may require 100 runs for each of nine alpha values (0.1-0.9), i.e. 900 runs total
- ELVAR is more sensitive to false positives in the clustering attribute (sample/phenotype label), which can affect community quality

## Evidence

- [results] assessed via one-sided Wilcoxon rank sum tests on the negative-binomial-regression P-values over 100 runs: [evidence span withheld — anchor status: not_found]
- [methods] doDA call with replicate structure: "da.o <- DoDA(seu.o,sigcl.o,varDA="naive",varREP="mouse");"
- [abstract] ELVAR sensitivity improvement motivating the benchmark: "demonstrating that ELVAR improves the sensitivity to detect cell-type composition shifts in relation to aging, precancerous states and Covid-19 phenotypes"
- [discussion] alpha optimization across nine values: "to identify an optimal a parameter value may require a total of 100 runs for each of nine a parameter values, so a total of 900 runs"
