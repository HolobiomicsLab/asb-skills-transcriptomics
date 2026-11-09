---
name: cross-method-integration-score-aggregation
description: Use when comparing >=2 cell-type annotation or integration methods across
  multiple scRNA-seq datasets and you need standardized per-method metrics plus a
  statistical verdict (paired t-tests, p < 0.05).
when_to_use_negative:
- Only one method on a single dataset is being evaluated — paired t-tests across datasets
  and score aggregation require multiple methods and replicate datasets
- Query data contains unseen cell types absent from the reference labels, so macro
  F1 against reference-defined categories is not meaningful without combining MASI
  with SCCAF
- Cell-type annotation naming/resolution is not standardized across the methods being
  compared, making label-based metrics incomparable
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MASI
  role: Method under test; produces gene2cell scores and MACA-transferred labels (marker_identification,
    parallel batch_size=50000) whose metrics feed the aggregation
  repo: https://github.com/hayatlab/MASI
- name: scNym
  role: Baseline label-transfer method run with its default pipeline before metric
    computation
  repo: https://github.com/calico/scnym
- name: Symphony
  role: Baseline reference-mapping method (Harmony-corrected reference build) benchmarked
    with the same metrics
  repo: https://github.com/immunogenomics/symphony
- name: Scanpy
  role: Underlying framework for dataset loading and preprocessing (pp.highly_variable_genes)
    prior to scoring
- name: MACA
  role: Performs MASI label transfer over-clustering (resolutions 3, 5, 7; neighborhood
    sizes 5, 10, 15) whose outputs are scored
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:08f37e540edfc4bbeeed1c3818b18b0c25fa7cfef1c99871b095f3184ef325be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hayatlab/MASI
  tools:
  - Harmony
  - LIGER
  - MACA
  - MASI
  - Scanorama
  - Scanpy
  - Seurat
  - Symphony
  - scArches
  - scNym
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# Cross-method integration score aggregation

## Summary

Benchmarked evaluation pipeline that scores each cell-annotation/integration method (e.g., MASI, SVM, scNym, scANVI, Symphony, Scanorama, LIGER, Seurat) per dataset on macro F1, overall accuracy, cell-type silhouette, and batch entropy, aggregates them into annotation/integration/overall scores, and uses paired t-tests across datasets to rank methods.

## When to use

Use when comparing >=2 cell-type annotation or integration methods across multiple scRNA-seq datasets and you need standardized per-method metrics plus a statistical verdict (paired t-tests, p < 0.05).

## When NOT to use

- Only one method on a single dataset is being evaluated — paired t-tests across datasets and score aggregation require multiple methods and replicate datasets
- Query data contains unseen cell types absent from the reference labels, so macro F1 against reference-defined categories is not meaningful without combining MASI with SCCAF
- Cell-type annotation naming/resolution is not standardized across the methods being compared, making label-based metrics incomparable

## Inputs

- Six processed scRNA-seq benchmark datasets (Anndata/h5ad) from figshare deposit 18866264
- Reference AnnData with cell-type labels in the 'cell_type' column
- Per-cell predicted labels from each method (MASI gene2cell labels, SVM predictions, scNym, scANVI, Symphony, Scanorama, LIGER, Seurat)
- Batch identifiers per cell for batch entropy computation

## Outputs

- Per-method, per-dataset metric table: macro F1, overall accuracy, cell-type silhouette, batch entropy mixing
- Aggregated annotation, integration, and overall performance scores per method
- Paired t-test p-values comparing MASI against each baseline
- benchmark_results.csv containing all metrics and p-values

## How to apply

For each method (MASI with gene2cell scores via masi.parallel(batch_size=50000), linear/non-linear SVM trained on the same cell-type markers MASI uses, scNym and scArches scANVI with default pipelines, Symphony with variable gene selection/scaling/PCA/Harmony reference building, Scanorama, LIGER, Seurat) run label transfer or integration on the same six benchmark datasets from the figshare deposit 18866264. Per method and dataset, compute macro F1 and overall accuracy for annotation quality, cell-type silhouette for integration compactness, and batch entropy mixing for integration mixing. Aggregate the four metrics into annotation, integration, and overall performance scores per method. Then perform paired t-tests comparing MASI against each baseline across the six datasets (significance at p < 0.05), and write all metrics and p-values to benchmark_results.csv. Rationale: all methods are trained/evaluated on identical data and identical marker sets so that metric differences are attributable to the method, not the inputs.

## Related tools

- **MASI** (Method under test; produces gene2cell scores and MACA-transferred labels (marker_identification, parallel batch_size=50000) whose metrics feed the aggregation) — https://github.com/hayatlab/MASI
- **scNym** (Baseline label-transfer method run with its default pipeline before metric computation) — https://github.com/calico/scnym
- **Symphony** (Baseline reference-mapping method (Harmony-corrected reference build) benchmarked with the same metrics) — https://github.com/immunogenomics/symphony
- **Scanpy** (Underlying framework for dataset loading and preprocessing (pp.highly_variable_genes) prior to scoring)
- **MACA** (Performs MASI label transfer over-clustering (resolutions 3, 5, 7; neighborhood sizes 5, 10, 15) whose outputs are scored)

## Examples

```
from scipy.stats import ttest_rel; pvals = {m: ttest_rel(masi_scores, df[m]) for m in baselines}; df.to_csv('benchmark_results.csv')
```

## Evaluation signals

- benchmark_results.csv contains one row per method per benchmark dataset with all four metrics (macro F1, ACC, silhouette, batch entropy) populated
- Each MASI-vs-baseline comparison has a paired t-test p-value, and all p-values except Seurat are < 0.05
- Overall performance score per method is reproducibly derivable from the aggregated annotation and integration score columns
- All methods are evaluated on the same six datasets and (for SVM) the same cell-type marker sets used by MASI — check input parity before trusting metric deltas
- Expected direction: MASI's overall score is significantly higher than all baselines except Seurat

## Limitations

- Conclusions hold only on the six datasets used in this study; scNym and scArches were run with default pipelines rather than tuned parameters, which may understate their performance
- No software version numbers are reported for any benchmarked tool, hindering exact reproduction
- MASI cannot annotate unseen cell types not present in the reference, which limits annotation metrics when the reference resolution does not match the query

## Evidence

- [abstract] We benchmark MASI with other well-established methods and demonstrate that MASI outperforms other methods, in terms of integration, annotation, and speed.: "We benchmark MASI with other well-established methods and demonstrate that MASI outperforms other methods, in terms of integration, annotation, and speed."
- [results] Paired t-tests across six datasets, p < 0.05 vs all methods except Seurat: [evidence span withheld — anchor status: not_found]
- [discussion] Same marker set used for SVM classifiers ensures fair comparison: "we used the same cell-type markers that were used for MASI to train both linear and non-linear SVM classifiers"
- [discussion] Default pipelines used for scNym/scArches benchmarking: "an optimal outcome for a specific data might require customized parameter tuning. However, for benchmarking, we used default pipelines of both methods"
- [readme] MASI marker identification and MACA label transfer: "MASI utilizes robust marker idenfication to identify marker genes from reference data and transfers cell-type labels to target data through MACA."
