---
name: group-wise-median-comparison-across-tools
description: Use when a supplementary table (e.g. Dataset EV2) lists a numeric parameter
  per study along with the tool that produced it, and the goal is to test whether
  the parameter differs between tool families.
when_to_use_negative:
- The numeric quantity is already aggregated per tool group (no per-study rows to
  group by tool annotation)
- The grouping variable is continuous rather than a categorical tool label, making
  median-by-group comparison inappropriate
- Graining-level or dataset-size columns contain unparseable/free-text values that
  cannot be type-cast without external curation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetacellAnalysisToolkit
  role: Toolkit whose construction tools (MetaCell2, SEACells, SuperCell) define the
    grouping categories for the graining-level comparison across studies in Dataset
    EV2
  repo: https://github.com/GfellerLab/MetacellAnalysisToolkit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s44320-024-00045-6
    title: Building and analyzing metacells in single-cell genomics data
  source_content_hash: sha256:36962239e5ba0b1d6528baf367648606d8cb79b6baba71406f11eede36f4f09a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GfellerLab/MetacellAnalysisToolkit
  tools:
  - MetacellAnalysisToolkit
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# group-wise-median-comparison-across-tools

## Summary

Statistical skill for comparing a numeric parameter (here, metacell graining level) across groups defined by the construction tool used, using medians, ranges, and rank correlation against a covariate such as dataset size, as demonstrated on the article's supplementary Dataset EV2.

## When to use

Use when a supplementary table (e.g. Dataset EV2) lists a numeric parameter per study along with the tool that produced it, and the goal is to test whether the parameter differs between tool families.

## When NOT to use

- The numeric quantity is already aggregated per tool group (no per-study rows to group by tool annotation)
- The grouping variable is continuous rather than a categorical tool label, making median-by-group comparison inappropriate
- Graining-level or dataset-size columns contain unparseable/free-text values that cannot be type-cast without external curation

## Inputs

- Supplementary Dataset EV2 table (studies using metacells, construction tool annotation, dataset sizes, graining levels), as CSV/Excel
- Grouping column: metacell construction tool name (MetaCell vs other tools)
- Numeric columns: graining level and dataset size per study

## Outputs

- ev2_graining_stats.csv with one row per statistic (overall median, min/max range, per-tool medians, Spearman correlation coefficient and p-value)
- Group-level median summary (MetaCell ~90 vs other tools ~55) and overall range (~5 to >1000, median ~80)

## How to apply

Load the supplementary table (Dataset EV2: studies using metacells, their construction tools, dataset sizes, and graining levels) into a pandas or R data frame and type-cast the graining-level and dataset-size columns, keeping each study's tool annotation. Compute the overall distribution of graining levels (minimum ~5, maximum >1000, median ~80) to characterize typical practice. Split studies by construction tool and compare group medians: MetaCell-algorithm studies (median ~90) versus studies using other metacell construction tools (median ~55). Compute a Spearman rank correlation between dataset size and graining level to verify the reported absence of correlation, recording both the coefficient and p-value. Write all statistics to a one-row-per-statistic CSV (ev2_graining_stats.csv) and validate each value against the article's reported figures.

## Related tools

- **MetacellAnalysisToolkit** (Toolkit whose construction tools (MetaCell2, SEACells, SuperCell) define the grouping categories for the graining-level comparison across studies in Dataset EV2) — https://github.com/GfellerLab/MetacellAnalysisToolkit

## Examples

```
df = pd.read_csv('Dataset_EV2.tsv', sep='\t'); df.groupby('tool')['graining_level'].median(); spearmanr(df['dataset_size'], df['graining_level'])
```

## Evaluation signals

- Overall graining-level statistics reproduce the article's reported figures: minimum ~5, maximum >1000, median ~80
- Per-tool medians match reported values: MetaCell median ~90 vs other-tools median ~55
- Spearman correlation between dataset size and graining level is non-significant, matching the reported absence of correlation
- ev2_graining_stats.csv contains exactly one row per statistic with the expected statistic names (overall median, range, per-tool medians, correlation coefficient, p-value)
- Every study row in the cleaned data frame retains its construction-tool annotation after type-casting (no dropped studies)

## Limitations

- Graining-level values are self-reported by surveyed studies, so the distribution (~5 to >1000) reflects heterogeneous reporting rather than a controlled measurement
- Median comparison across tools is descriptive; the article reports the medians (~90 vs ~55) and absence of correlation with dataset size but no formal significance test for the group difference

## Evidence

- [results] Graining levels used across metacell studies ranged from very small (~5) to very large (>1000) with a median of around 80; the MetaCell algorithm resulted in larger graining levels (median 90) compared to other methods (median 55).: [evidence span withheld — anchor status: not_found]
- [results] the MetaCell algorithm resulted in larger graining levels (median 90) compared to other methods (median 55): [evidence span withheld — anchor status: not_found]
- [supplementary] Compute a rank correlation (e.g. Spearman) between dataset size and graining level across studies to verify the reported absence of correlation.: [evidence span withheld — anchor status: not_found]
- [abstract] a fully integrated pipeline to rapidly build, visualize and evaluate metacells with different methods ( https://github.com/GfellerLab/MetacellAnalysisToolkit ): "a fully integrated pipeline to rapidly build, visualize and evaluate metacells with different methods"
