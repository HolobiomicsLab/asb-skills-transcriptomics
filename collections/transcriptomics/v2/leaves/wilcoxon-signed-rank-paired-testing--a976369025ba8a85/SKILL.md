---
name: wilcoxon-signed-rank-paired-testing
description: Use when two methods are each evaluated on the same paired set of datasets
  (e.g., 44 atlas-scale normal/cancer tissues with per-dataset F1, balanced accuracy,
  or overall accuracy) and a nonparametric significance test of the metric differences
  is required.
when_to_use_negative:
- The two methods were evaluated on different, non-paired datasets — the signed-rank
  test requires pairwise differences on identical observations; use an unpaired test
  instead
- Only very few paired observations (e.g., fewer than ~6 datasets) are available,
  so signed-rank inference is underpowered
- The outcome of interest is runtime throughput alone, where the article's significant
  differences favored scType and the paired test should be reported rather than assumed
  favorable to Census
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: ggpubr
  role: Provides stat_compare_means / Wilcoxon signed-rank comparisons of overall
    accuracy between Census and scType across the 44 datasets and publication-ready
    paired-comparison plots
  repo: github.com/kassambara/ggpubr
- name: Census
  role: Method under evaluation; its hierarchical gradient-boosted decision tree annotations
    supply the per-dataset accuracy values tested against scType
  repo: github.com/sjdlabgroup/Census
- name: Seurat
  role: Seurat v4 loads and preprocesses the 44 held-out datasets (NormalizeData,
    ScaleData, RunPCA, RunUMAP) so paired per-dataset annotations exist for both methods
  repo: github.com/sjdlabgroup/Census
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:5dd749ab8600790597ed8ae7f987485f4cf694748d405eeeb819e5c16b66da9d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Census
  - Seurat
  - ggpubr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# wilcoxon-signed-rank-paired-testing

## Summary

Nonparametric paired Wilcoxon signed-rank testing of per-dataset metric differences between two annotation methods (e.g., Census vs scType overall accuracy across 44 held-out human/mouse tissue datasets), implemented in R with ggpubr's stat_compare_means.

## When to use

Use when two methods are each evaluated on the same paired set of datasets (e.g., 44 atlas-scale normal/cancer tissues with per-dataset F1, balanced accuracy, or overall accuracy) and a nonparametric significance test of the metric differences is required.

## When NOT to use

- The two methods were evaluated on different, non-paired datasets — the signed-rank test requires pairwise differences on identical observations; use an unpaired test instead
- Only very few paired observations (e.g., fewer than ~6 datasets) are available, so signed-rank inference is underpowered
- The outcome of interest is runtime throughput alone, where the article's significant differences favored scType and the paired test should be reported rather than assumed favorable to Census

## Inputs

- Per-dataset paired metric table: one overall accuracy/F1/balanced accuracy/label-similarity value per method (Census, scType) per held-out dataset (44 datasets)
- Census per-cell annotations produced via census_main on Seurat v4 objects with UMAP reductions
- scType per-cell annotations on the same datasets
- Harmonized ground-truth cell-type labels (105 labels across 1,769,071 cells)

## Outputs

- Wilcoxon signed-rank p-values for paired metric comparisons (reported as all P < 2e−16 for accuracy/F1/balanced accuracy/label similarity)
- Significance calls per metric (accept at p < 0.05)
- benchmark_metrics.csv with per-dataset metrics for both methods

## How to apply

Structure the benchmark so both methods (Census and scType) annotate the identical 44 held-out datasets (1,769,071 cells, 105 harmonized labels), producing one metric value per dataset per method so the observations are paired. Compute per-dataset metrics (F1, balanced accuracy, label similarity, overall accuracy) against the harmonized ground-truth labels before testing. Run the Wilcoxon signed-rank test on the paired per-dataset values — e.g., with ggpubr's stat_compare_means(method = 'wilcox.test', paired = TRUE) on the paired overall accuracies — and require p < 0.05 to accept significance. Report the achieved p-values (this benchmark reported all Wilcoxon P < 2e−16 favoring Census on F1, balanced accuracy, label similarity, and overall accuracy, while scType had shorter run times) and write per-dataset metrics to benchmark_metrics.csv.

## Related tools

- **ggpubr** (Provides stat_compare_means / Wilcoxon signed-rank comparisons of overall accuracy between Census and scType across the 44 datasets and publication-ready paired-comparison plots) — github.com/kassambara/ggpubr
- **Census** (Method under evaluation; its hierarchical gradient-boosted decision tree annotations supply the per-dataset accuracy values tested against scType) — github.com/sjdlabgroup/Census
- **Seurat** (Seurat v4 loads and preprocesses the 44 held-out datasets (NormalizeData, ScaleData, RunPCA, RunUMAP) so paired per-dataset annotations exist for both methods) — github.com/sjdlabgroup/Census

## Examples

```
ggpubr::stat_compare_means(method = "wilcox.test", paired = TRUE, comparisons = list(c("Census", "scType")))
```

## Evaluation signals

- Pairing check: every one of the 44 datasets contributes exactly one accuracy value per method, so differences are computed within datasets
- Significance threshold: each paired metric comparison must yield Wilcoxon p < 0.05 (article reported P < 2e−16) to accept Census outperforms scType
- Directionality: F1, balanced accuracy, label similarity, and overall accuracy must favor Census, while runtime comparison must show scType shorter — check the sign of paired differences per metric
- Subtype tally coherence: paired metric superiority should be consistent with Census identifying 81/105 subtypes vs scType's 35/89
- Output schema: benchmark_metrics.csv contains per-dataset rows with both methods' F1, balanced accuracy, label similarity, overall accuracy, and runtime

## Limitations

- The Wilcoxon signed-rank test is nonparametric and rank-based; it tests median paired differences and does not quantify effect size magnitude on its own
- Runtime was significantly better for scType, so paired significance does not uniformly favor Census across all metrics
- The test is valid only under the paired design used in the article; substituting unpaired data invalidates the comparison

## Evidence

- [abstract] Benchmarked on 44 atlas-scale normal and cancer, human and mouse tissues, Census significantly outperforms state-of-the-art methods across multiple metrics: [evidence span withheld — anchor status: not_found]
- [full_text] Run Wilcoxon signed-rank comparisons of overall accuracy between Census and scType across the 44 datasets using ggpubr.: [evidence span withheld — anchor status: not_found]
- [readme] The ‘ggpubr’ package provides some easy-to-use functions for creating and customizing ‘ggplot2’-based publication ready plots.: [evidence span withheld — anchor status: external_doc]
- [readme] Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R.: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
