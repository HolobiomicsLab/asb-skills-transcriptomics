---
name: metacell-construction-tool-taxonomy
description: Use when reviewing or benchmarking metacell construction tools and you
  need to quantify how coarse or fine published metacell analyses were grained, or
  compare target metacell sizes across tools (MetaCell vs SuperCell, SEACells, etc.).
when_to_use_negative:
- You are building metacells for a new dataset rather than surveying how published
  studies sized their metacells — use the MetacellAnalysisToolkit pipeline instead.
- Your input is raw scRNA-seq count data rather than the curated study-level Dataset
  EV2 table.
- You need per-metacell quality metrics (purity, compactness, separation, INV) rather
  than population-level graining statistics across studies.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0091
tools:
- name: MetacellAnalysisToolkit
  role: Toolkit accompanying the review; provides the metacell construction methods
    (SEACells, SuperCell, MetaCell2) whose graining levels are being compared across
    studies
  repo: https://github.com/GfellerLab/MetacellAnalysisToolkit
- name: SuperCell
  role: Metacell construction tool in the 'other tools' comparison group against MetaCell
  repo: https://github.com/GfellerLab/SuperCell
- name: SEACells
  role: Metacell construction tool in the 'other tools' comparison group against MetaCell
  repo: https://github.com/dpeerlab/SEACells
- name: metacells
  role: Python metacell construction tool in the 'other tools' comparison group against
    MetaCell
  repo: https://github.com/tanaylab/metacells
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s44320-024-00045-6
    title: Building and analyzing metacells in single-cell genomics data
  source_content_hash: sha256:ad020eb1596f3087bd5fc0ece8acc592f570e694e885933120cc3a92ca2da25f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GfellerLab/MetacellAnalysisToolkit
  tools:
  - MetacellAnalysisToolkit
  - SEACells
  - SuperCell
  - metacells
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metacell-construction-tool-taxonomy

## Summary

A survey/analysis skill for characterizing the range of graining levels (metacell sizes) used across published single-cell studies and comparing graining behavior between the MetaCell tool and other metacell construction tools, using the article's supplementary Dataset EV2.

## When to use

Use when reviewing or benchmarking metacell construction tools and you need to quantify how coarse or fine published metacell analyses were grained, or compare target metacell sizes across tools (MetaCell vs SuperCell, SEACells, etc.).

## When NOT to use

- You are building metacells for a new dataset rather than surveying how published studies sized their metacells — use the MetacellAnalysisToolkit pipeline instead.
- Your input is raw scRNA-seq count data rather than the curated study-level Dataset EV2 table.
- You need per-metacell quality metrics (purity, compactness, separation, INV) rather than population-level graining statistics across studies.

## Inputs

- Supplementary Dataset EV2 table (studies using metacells, their construction tools, dataset sizes, and graining levels)
- Tabular data frame readable by pandas or R

## Outputs

- ev2_graining_stats.csv with one row per statistic (overall median, min/max range, per-tool medians, Spearman correlation coefficient and p-value)
- Graining-level distribution summary (min ~5, max >1000, median ~80)
- Per-tool median graining levels (MetaCell ~90 vs other tools ~55)

## How to apply

Load the supplementary Dataset EV2 table, which catalogs studies using metacells together with their construction tool, dataset size, and graining level, into a pandas or R data frame. Clean and type-cast the graining-level and dataset-size columns while preserving each study's tool annotation. Compute the overall graining-level distribution (reported range ~5 to >1000, median ~80), then split studies by construction tool and compute per-tool medians to contrast MetaCell-based studies (median ~90) with studies using other tools (median ~55). Optionally compute a Spearman rank correlation between dataset size and graining level to verify the reported absence of correlation, and write all statistics to ev2_graining_stats.csv with one row per statistic.

## Related tools

- **MetacellAnalysisToolkit** (Toolkit accompanying the review; provides the metacell construction methods (SEACells, SuperCell, MetaCell2) whose graining levels are being compared across studies) — https://github.com/GfellerLab/MetacellAnalysisToolkit
- **SuperCell** (Metacell construction tool in the 'other tools' comparison group against MetaCell) — https://github.com/GfellerLab/SuperCell
- **SEACells** (Metacell construction tool in the 'other tools' comparison group against MetaCell) — https://github.com/dpeerlab/SEACells
- **metacells** (Python metacell construction tool in the 'other tools' comparison group against MetaCell) — https://github.com/tanaylab/metacells

## Examples

```
Rscript -e "ev2 <- read.csv('Dataset_EV2.csv'); stats <- data.frame(stat=c('median_all','min','max','median_metacell','median_other'), value=c(median(ev2$graining), min(ev2$graining), max(ev2$graining), median(ev2$graining[ev2$tool=='MetaCell']), median(ev2$graining[ev2$tool!='MetaCell']))); write.csv(stats, 'ev2_graining_stats.csv', row.names=FALSE)"
```

## Evaluation signals

- Overall graining-level distribution matches the article: minimum ~5, maximum >1000, median ~80
- Per-tool medians reproduce the reported contrast: MetaCell-based studies median ~90 vs other-tools median ~55
- Spearman correlation between dataset size and graining level is non-significant, consistent with the reported absence of correlation
- Every study row retains its construction-tool annotation after cleaning (no dropped or NA tool labels)
- ev2_graining_stats.csv contains exactly one row per statistic with correlation coefficient and p-value populated

## Limitations

- Graining levels and tool assignments come from a literature survey (Dataset EV2) compiled by manual expert review, so values reflect published practice, not controlled benchmarking.
- Median graining levels are descriptive summaries; they do not establish that one tool produces better-sized metacells.
- The reported ranges (~5 to >1000) are approximate figures validated against the article's text, not exact recomputed bounds.

## Evidence

- [supplementary] Graining levels used across metacell studies ranged from very small (~5) to very large (>1000) with a median of around 80; the MetaCell algorithm resulted in larger graining levels (median 90) compared to other methods (median 55).: [evidence span withheld — anchor status: not_found]
- [supplementary] Dataset EV2 catalog of studies: [evidence span withheld — anchor status: not_found]
- [abstract] Metacells reduce size and complexity: "metacells have been introduced to reduce the size and complexity of single-cell genomics data while preserving biologically relevant information and improving interpretability"
- [readme] metacell concept rationale: [evidence span withheld — anchor status: external_doc]
