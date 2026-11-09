---
name: wilcoxon-signed-rank-significance-testing
description: Use when comparing two methods' per-sample performance metrics (e.g.,
  CCC/Pearson/RMSE over the same 124 pseudobulk samples) and a paired, one-sided significance
  assessment of whether one method outperforms the other is required.
when_to_use_negative:
- Metric vectors are unpaired (different samples per method), where a paired signed-rank
  test is invalid and an unpaired test would be required
- The comparison involves more than two related conditions simultaneously, requiring
  a different omnibus procedure rather than pairwise Wilcoxon tests
- The analysis goal is deconvolution itself rather than significance assessment of
  already-computed metric differences
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SciPy
  role: Provides the one-sided Wilcoxon signed-rank test used to assess pairwise statistical
    significance between deconvolution methods, and the Pearson correlation coefficient
    used to compute the per-sample metric vectors fed into the test
- name: scSemiProfiler
  role: Method under test whose inferred cell-type proportions (from 4- and 28-representative
    references) are compared against competing deconvolution methods in the Wilcoxon
    tests
  repo: https://github.com/mcgilldinglab/scSemiProfiler
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:b90b5f3d97f59c87b49d86b04fadbe108fb82d1f97ba60640a5e30965f2267bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - Bisque
  - CIBERSORTx
  - EPIC
  - MuSiC
  - NNLS
  - Scaden
  - SciPy
  - Scikit-learn
  - TAPE
  - scSemiProfiler
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# wilcoxon-signed-rank-significance-testing

## Summary

One-sided Wilcoxon signed-rank test implemented in SciPy, used in the scSemiProfiler COVID-19 deconvolution benchmark to assess pairwise statistical significance of deconvolution accuracy differences (Lin's CCC, Pearson correlation, RMSE) between methods across the 124 pseudobulk samples.

## When to use

Use when comparing two methods' per-sample performance metrics (e.g., CCC/Pearson/RMSE over the same 124 pseudobulk samples) and a paired, one-sided significance assessment of whether one method outperforms the other is required.

## When NOT to use

- Metric vectors are unpaired (different samples per method), where a paired signed-rank test is invalid and an unpaired test would be required
- The comparison involves more than two related conditions simultaneously, requiring a different omnibus procedure rather than pairwise Wilcoxon tests
- The analysis goal is deconvolution itself rather than significance assessment of already-computed metric differences

## Inputs

- Per-sample metric vectors (Lin's CCC, Pearson correlation, RMSE) for each deconvolution method, aligned by pseudobulk sample across the 124-sample COVID-19 cohort
- Ground-truth cell type proportions per pseudobulk sample (as derived from annotated single-cell data)

## Outputs

- Pairwise one-sided Wilcoxon p-value columns in deconvolution_benchmark.csv (one row per method-condition, 9 rows: scSemiProfiler-28, scSemiProfiler-4, and 7 competing methods)
- Significance assessments supporting pairwise comparisons of deconvolution performance for Fig. 2j-l

## How to apply

After computing each method's estimated cell-type proportions on the 124-sample COVID-19 pseudobulk cohort and deriving per-sample error/agreement metrics (Lin's CCC, Pearson correlation via SciPy, RMSE) against ground-truth proportions, form paired vectors of metric values for each pair of methods (scSemiProfiler-28, scSemiProfiler-4, and the 7 competitors: CIBERSORTx, Bisque, Scaden, TAPE, EPIC, NNLS, MuSiC). Because the same samples are evaluated by both methods in each pair, use the paired (signed-rank) Wilcoxon test, not an unpaired rank-sum test, and specify a one-sided alternative in SciPy reflecting the hypothesized superiority direction. Record each pairwise p-value as a column in deconvolution_benchmark.csv alongside CCC, Pearson, and RMSE, yielding one row per method-condition (9 rows total). Validate that the file matches the article's Source Data for Fig. 2j-l (doi 10.1038/s41467-024-50150-1).

## Related tools

- **SciPy** (Provides the one-sided Wilcoxon signed-rank test used to assess pairwise statistical significance between deconvolution methods, and the Pearson correlation coefficient used to compute the per-sample metric vectors fed into the test)
- **scSemiProfiler** (Method under test whose inferred cell-type proportions (from 4- and 28-representative references) are compared against competing deconvolution methods in the Wilcoxon tests) — https://github.com/mcgilldinglab/scSemiProfiler

## Examples

```
from scipy.stats import wilcoxon; stat, p = wilcoxon(a_metrics, b_metrics, alternative='greater')
```

## Evaluation signals

- deconvolution_benchmark.csv exists with exactly 9 method rows (scSemiProfiler-28, scSemiProfiler-4, and the 7 competing methods)
- Each row carries CCC, Pearson correlation, RMSE, and one-sided Wilcoxon p-value columns
- Metric values and p-values match the article's Source Data for Fig. 2j-l (doi 10.1038/s41467-024-50150-1)
- The Wilcoxon test is applied as paired, one-sided comparisons between aligned per-sample metric vectors, not as unpaired tests
- Findings agree with the article's claim that scSemiProfiler outperformed all benchmarked methods under both reference conditions

## Limitations

- Significance conclusions depend on the quality of the upstream deconvolution estimates; conventional deconvolution methods are constrained to cell-type-level resolution and often show low accuracy on real bulk data
- scSemiProfiler comparisons require a single-cell reference; the fair-comparison condition confines the reference to the first 4 representatives to match other methods' capacity
- No software versions for scSemiProfiler or dependencies are stated, so exact numeric reproduction may be sensitive to dependency versions

## Evidence

- [methods] Assess pairwise statistical significance between methods with a one-sided Wilcoxon test implemented in SciPy.: [evidence span withheld — anchor status: not_found]
- [results] In the COVID-19 cohort deconvolution benchmark (all N = 124 samples, evaluated via CCC, Pearson correlation and RMSE), scSemiProfiler demonstrated superior deconvolution performance over all benchmarked methods.: [evidence span withheld — anchor status: not_found]
- [other] Write deconvolution_benchmark.csv containing one row per method-condition (scSemiProfiler-28, scSemiProfiler-4, and the 7 competing methods = 9 rows).: [evidence span withheld — anchor status: not_found]
- [discussion] The tool's deep generative model enables single-cell level deconvolution across various studies, provided there is a single-cell reference.: "The tool’s deep generative model enables “single-cell level deconvolution” across various studies, provided there is a single-cell reference."
