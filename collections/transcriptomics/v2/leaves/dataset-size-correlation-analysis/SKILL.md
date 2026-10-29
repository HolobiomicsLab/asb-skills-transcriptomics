---
name: dataset-size-correlation-analysis
description: Use when a supplementary survey table lists studies with numeric attributes
  (e.g., dataset size, graining level) and a categorical tool/method annotation, and
  the goal is distribution summaries plus a size-vs-parameter rank correlation.
when_to_use_negative:
- Input is single-cell or metacell expression data (counts, h5ad, Seurat objects)
  rather than a per-study survey table — use the metacell QC/computation skills instead
- The relationship between size and the target variable may be non-monotonic and needs
  flexible regression, since Spearman only detects monotonic rank association
- The table lacks per-study construction-tool annotations, so the per-tool median
  comparison is impossible
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3316
tools:
- name: MetacellAnalysisToolkit
  role: Toolkit context providing the metacell construction tools (MetaCell, SEACells,
    SuperCell, MetaCell2) whose usage patterns are tabulated across studies in Dataset
    EV2
  repo: https://github.com/GfellerLab/MetacellAnalysisToolkit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s44320-024-00045-6
    title: Building and analyzing metacells in single-cell genomics data
  source_content_hash: sha256:a7882ef6847c37053ed016afc003bf64280f3b5b584203e6279394995b21dad6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GfellerLab/MetacellAnalysisToolkit
  tools:
  - MetacellAnalysisToolkit
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# dataset-size-correlation-analysis

## Summary

Statistical re-analysis of supplementary study-survey tables (e.g., Dataset EV2 from the metacell review by Bilous et al.) to characterize graining-level distributions across studies and test whether graining level correlates with dataset size, split by metacell construction tool.

## When to use

Use when a supplementary survey table lists studies with numeric attributes (e.g., dataset size, graining level) and a categorical tool/method annotation, and the goal is distribution summaries plus a size-vs-parameter rank correlation.

## When NOT to use

- Input is single-cell or metacell expression data (counts, h5ad, Seurat objects) rather than a per-study survey table — use the metacell QC/computation skills instead
- The relationship between size and the target variable may be non-monotonic and needs flexible regression, since Spearman only detects monotonic rank association
- The table lacks per-study construction-tool annotations, so the per-tool median comparison is impossible

## Inputs

- Supplementary survey table of studies (Dataset EV2) with columns for dataset size, graining level, and metacell construction tool
- Reported statistics from the article for validation (graining range ~5 to >1000, median ~80; MetaCell median ~90 vs other-tools median ~55)

## Outputs

- ev2_graining_stats.csv with one row per statistic: overall median, range, per-tool medians, Spearman correlation coefficient and p-value

## How to apply

Load the supplementary table (Dataset EV2) into a pandas or R data frame, then clean and type-cast the graining-level and dataset-size columns while retaining each study's construction-tool annotation. Compute the overall graining-level distribution (minimum ~5, maximum >1000, median ~80) and split studies by construction tool to compare median graining levels for MetaCell-based studies versus other tools (expected ~90 vs ~55). Finally compute a Spearman rank correlation between dataset size and graining level across studies to verify the reported absence of correlation, and write all statistics (overall median, range, per-tool medians, correlation coefficient and p-value) to ev2_graining_stats.csv with one row per statistic.

## Related tools

- **MetacellAnalysisToolkit** (Toolkit context providing the metacell construction tools (MetaCell, SEACells, SuperCell, MetaCell2) whose usage patterns are tabulated across studies in Dataset EV2) — https://github.com/GfellerLab/MetacellAnalysisToolkit

## Examples

```
df = pd.read_excel('Dataset_EV2.xlsx'); stats = {'overall_median': df.graining.median(), 'range': (df.graining.min(), df.graining.max()), 'metafish_vs_other_medians': df.groupby('tool').graining.median(), 'spearman': scipy.stats.spearmanr(df.dataset_size, df.graining)}
```

## Evaluation signals

- Overall graining-level statistics reproduce the article's reported figures: minimum ~5, maximum >1000, median ~80
- Splitting by construction tool yields median graining level ~90 for MetaCell-based studies versus ~55 for other tools
- Spearman correlation between dataset size and graining level is not statistically significant, consistent with the reported absence of correlation
- ev2_graining_stats.csv contains exactly one row per statistic (overall median, range, per-tool medians, correlation coefficient, p-value)
- Graining-level and dataset-size columns pass type-casting checks (numeric dtypes, no malformed values) with tool annotations preserved

## Limitations

- Results depend on faithful extraction and type-casting of Dataset EV2; graining levels are self-reported by surveyed studies and may be inconsistently defined across the many nomenclature variants of the metacell concept
- Spearman rank correlation cannot capture non-monotonic associations between dataset size and graining level
- Only correlation with dataset size was tested; other covariates (e.g., publication year, data modality) are not assessed by this skill

## Evidence

- [supplementary] Graining levels used across metacell studies ranged from very small (~5) to very large (>1000) with a median of around 80: [evidence span withheld — anchor status: not_found]
- [supplementary] MetaCell algorithm resulted in larger graining levels (median 90) compared to other methods (median 55): [evidence span withheld — anchor status: not_found]
- [supplementary] Load the supplementary Dataset EV2 table into a pandas or R data frame: [evidence span withheld — anchor status: not_found]
- [supplementary] Compute a rank correlation between dataset size and graining level: [evidence span withheld — anchor status: not_found]
- [readme] reduction in the number of cells to analyze when dealing with large data: [evidence span withheld — anchor status: external_doc]
