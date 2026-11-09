---
name: statistical-significance-testing
description: Use when comparing method performance across multiple scRNA-seq benchmark
  datasets and you must show the winner's advantage is statistically significant,
  e.g. via one-sided Wilcoxon tests on per-dataset scores.
when_to_use_negative:
- Scores are unpaired (different datasets per method) — a paired Wilcoxon test would
  be invalid
- The evaluation goal is ranking embeddings visually rather than testing statistical
  significance of a score difference
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0091
tools:
- name: scDHA
  role: Method under test whose scores (silhouette index, classification accuracy,
    ARI) are compared against competitors with significance tests
  repo: https://github.com/duct317/scDHA
- name: mclust
  role: R package used to compute the adjusted Rand Index between scDHA clusters and
    ground-truth labels
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:7a81ff1aba3e4c2aa975e13cb76a7034a0640e6bebc49771edb62c374b6f8975
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scDHA
  tools:
  - Gradient Boosting Machine (GBM)
  - H2O
  - Random Forest (RF)
  - mclust
  - scDHA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# statistical-significance-testing

## Summary

Apply one-sided paired significance tests (Wilcoxon signed-rank) and clustering-quality metrics (adjusted Rand index, silhouette index) to statistically validate that scDHA outperforms competing methods across benchmark scRNA-seq datasets.

## When to use

Use when comparing method performance across multiple scRNA-seq benchmark datasets and you must show the winner's advantage is statistically significant, e.g. via one-sided Wilcoxon tests on per-dataset scores.

## When NOT to use

- Scores are unpaired (different datasets per method) — a paired Wilcoxon test would be invalid
- The evaluation goal is ranking embeddings visually rather than testing statistical significance of a score difference

## Inputs

- Per-dataset silhouette index scores for each visualization method (scDHA.vis, PCA, t-SNE, UMAP, SCANPY) across 34 scRNA-seq datasets
- Per-pair classification accuracy for each classifier (scDHA.class, XGBoost, RF, DL, GBM) across 20 pancreas train-test combinations
- Clustering assignments and ground-truth cell-type labels (e.g. from the Goolam dataset)

## Outputs

- Per-dataset silhouette-index score table
- classification_accuracy_table.csv with accuracies for all five methods across 20 train-test pairs
- Adjusted Rand Index values comparing predicted clusters to reference labels
- Reported one-sided Wilcoxon test p-values (e.g. 1.7e-6, 2.1e-8) and average-score comparison figures

## How to apply

Collect per-dataset scores for each competing method: silhouette index of 2D embeddings for visualization comparisons (scDHA.vis vs PCA, t-SNE, UMAP, SCANPY on 34 scRNA-seq datasets) or per-pair classification accuracy for cross-dataset tests (scDHA.class vs XGBoost, RF, DL, GBM on 20 pancreas train-test pairs). Because the same datasets/pairs are evaluated under every method, use a paired one-sided Wilcoxon signed-rank test to assess whether scDHA's scores exceed each competitor's; the paper reports p = 1.7e-6 for visualization and p = 2.1e-8 for classification in scDHA's favor. For clustering benchmarks, quantify agreement with ground-truth labels using the adjusted Rand index via mclust::adjustedRandIndex. Report per-dataset score tables (e.g. silhouette-index table, classification_accuracy_table.csv) alongside averages and p-values so the significance claim is reproducible.

## Related tools

- **scDHA** (Method under test whose scores (silhouette index, classification accuracy, ARI) are compared against competitors with significance tests) — https://github.com/duct317/scDHA
- **mclust** (R package used to compute the adjusted Rand Index between scDHA clusters and ground-truth labels)

## Examples

```
wilcox.test(scDHA_scores, competitor_scores, paired = TRUE, alternative = "greater")
```

## Evaluation signals

- The one-sided paired Wilcoxon test between scDHA and each competitor yields a significant p-value at the reported level (visualization: p = 1.7e-6; classification: p = 2.1e-8)
- scDHA attains the highest average silhouette index across all 34 datasets and highest accuracy across all 20 train-test pairs (average accuracy 0.96 vs 0.77 XGB, 0.69 RF, 0.43 DL, 0.72 GBM)
- Per-dataset/per-pair score tables are complete: 34 rows for visualization, 20 rows for classification, no missing method scores
- ARI values computed with mclust::adjustedRandIndex reproduce the reported clustering quality for seeded scDHA runs (seed = 1)

## Limitations

- One-sided Wilcoxon tests assume paired, roughly symmetric score differences; they establish superiority of scDHA over competitors but not effect size
- Accuracy claims are tied to the specific benchmark suites (34 visualization datasets, 5 pancreas datasets in 20 train-test pairs); the discussion notes generality claims to other domains (cancer, obesity, aging) are not supported by cited experiments
- Comparative claims against some alternatives (e.g. scVI) are deferred to supplementary materials (Supplementary Section 6 and Fig. 25–32)

## Evidence

- [abstract] scDHA outperforms state-of-the-art techniques in cell segregation through unsupervised learning, visualization of transcriptome landscape, cell classification, and pseudo-time inference: [evidence span withheld — anchor status: not_found]
- [full_text] Average the silhouette index per method across the 34 datasets and assess the difference between scDHA.vis and each competitor statistically: [evidence span withheld — anchor status: not_found]
- [results] the difference is statistically significant (p = 2.1 × 10−8, one-sided Wilcoxon test): [evidence span withheld — anchor status: not_found]
- [readme] Calculating adjusted Rand Index using mclust package: "Calculating adjusted Rand Index using mclust package: `mclust::adjustedRandIndex(cluster,label)`"
