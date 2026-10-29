---
name: roc-based-annotation-benchmarking
description: Use when you need to quantitatively compare two or more marker-based
  cell annotation methods on identical datasets, using author-curated cell labels
  as ground truth and AUROC as the comparison metric.
when_to_use_negative:
- The marker catalog includes gene sets derived from the very atlases being benchmarked
  — this introduces circularity and inflates AUROC; exclude such sets first.
- You lack author-curated (ground-truth) cell labels for the test datasets, making
  ROC evaluation against reference labels impossible.
- You only need to annotate a single new dataset rather than compare annotation methods
  head-to-head — use the Miko scoring pipeline directly instead.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMiko
  role: Provides nullScore()/mikoScore()/sigScore(), coherentFraction(), and benchmarkScores()/benchmarkCluster()
    for scoring, filtering, and ROC-based evaluation of annotations
  repo: https://github.com/NMikolajewicz/scMiko
- name: Seurat
  role: Framework for loading/preprocessing benchmark Seurat objects and providing
    the AddModuleScore baseline scoring method
- name: sc-type
  role: Baseline annotation method (scType) scored on the same shared marker catalog
    for comparison
  repo: https://github.com/IanevskiAleksandr/sc-type
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:be0a68b944f79b9714e57ff42c9b508fea9f6517fa2afe4b2935415c08720077
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - Seurat
  - sc-type
  - scMiko
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# roc-based-annotation-benchmarking

## Summary

Benchmark cell-type annotation methods (Miko scoring, Seurat AddModuleScore, SCINA, GSVA, scType) by scoring a shared marker catalog on the same datasets and evaluating each method's cluster-level annotations against author-curated labels via ROC/AUROC analysis using scMiko's benchmarkScores()/benchmarkCluster().

## When to use

Use when you need to quantitatively compare two or more marker-based cell annotation methods on identical datasets, using author-curated cell labels as ground truth and AUROC as the comparison metric.

## When NOT to use

- The marker catalog includes gene sets derived from the very atlases being benchmarked — this introduces circularity and inflates AUROC; exclude such sets first.
- You lack author-curated (ground-truth) cell labels for the test datasets, making ROC evaluation against reference labels impossible.
- You only need to annotate a single new dataset rather than compare annotation methods head-to-head — use the Miko scoring pipeline directly instead.

## Inputs

- Preprocessed Seurat objects (.rds) for benchmark datasets (Tyser 2021, Pijuan-Sala 2019, Cao 2020)
- Shared cell-type marker catalog (Cell_Catalog), excluding sets derived from benchmarked atlases
- Author-curated cell-type labels for each benchmark dataset

## Outputs

- Per-cluster cell-type annotations from each method (Miko scoring, AddModuleScore, SCINA, GSVA, scType)
- ROC curves and AUROC values per method against author-curated labels
- AUROC comparison table (benchmark_auroc_comparison.csv)

## How to apply

Load the preprocessed Seurat objects for all benchmark datasets (e.g., so_tyser2021_220621.rds, ps_count_matrix.rds/ps_meta_data.rds, and the Cao 2020 dataset) with the scMiko package. Score one shared marker catalog (Cell_Catalog) with every method — the Miko pipeline via nullScore() with n.replicate=10, nbin=24, gene set sizes 2–200 and step.size=10, then mikoScore() and sigScore(), and the baselines via Seurat AddModuleScore, SCINA, GSVA, and scType — excluding marker sets derived from the benchmarked atlases themselves to avoid circularity. For each method, assign cell types per cluster and apply the Miko post-scoring filters: fdr < 0.05, miko_score > 0, coherence_fraction >= 0.8 via coherentFraction(), and the frequent-flier flag at threshold 0.75. Finally, compute ROC curves and AUROC for each method's annotations against author-curated labels with scMiko's benchmarkScores()/benchmarkCluster(), passing the miko.fdr.threshold and coherence.fraction.threshold settings, and assemble the per-method AUROC values into a comparison table. In this benchmark, Miko scoring with post-hoc coherence filtering reached AUROC 0.776 versus GSVA (0.705), SCINA (0.603), and other baselines.

## Related tools

- **scMiko** (Provides nullScore()/mikoScore()/sigScore(), coherentFraction(), and benchmarkScores()/benchmarkCluster() for scoring, filtering, and ROC-based evaluation of annotations) — https://github.com/NMikolajewicz/scMiko
- **Seurat** (Framework for loading/preprocessing benchmark Seurat objects and providing the AddModuleScore baseline scoring method)
- **sc-type** (Baseline annotation method (scType) scored on the same shared marker catalog for comparison) — https://github.com/IanevskiAleksandr/sc-type

## Examples

```
devtools::install_github(repo = "NMikolajewicz/scMiko"); so.query <- readRDS("../data/demo/so_tyser2021_220621.rds")
```

## Evaluation signals

- AUROC for the Miko scoring method with post-hoc coherence filtering should be reported (in the paper: 0.776) and exceed baselines (GSVA 0.705, SCINA 0.603).
- All methods are evaluated on the same shared Cell_Catalog marker sets and same datasets, so per-method AUROC values must come from identical inputs (self-derived marker sets excluded).
- Miko annotations passed through filters before ROC computation: fdr < 0.05, miko_score > 0, coherence_fraction >= 0.8, frequent-flier threshold 0.75 — verify these thresholds were applied.
- Output comparison table (benchmark_auroc_comparison.csv) contains one AUROC row per benchmarked method.
- ROC curves computed via benchmarkScores()/benchmarkCluster() with miko.fdr.threshold and coherence.fraction.threshold set to the filter values.

## Limitations

- Marker-based annotations are only as good as the cell-type prescribed to the original dataset; no validation of source catalog curations is provided.
- Occasional co-similarities between marker sets from heterogeneous cell types reflect either marker curation inaccuracies or cross-cell-type process similarities — left unresolved.
- Benchmark results depend on the specific ground-truth atlases (Tyser 2021, Pijuan-Sala 2019, Cao 2020) and the shared Cell_Catalog; AUROC values may not transfer to other tissues or marker resources.

## Evidence

- [methods] Miko scoring pipeline steps and null model: "The miko scoring pipeline is a 3-step workflow that is implemented in nullScore(), mikoScore(), and sigScore() functions."
- [methods] Null model corrects gene set-size bias: "In step 1 we run nullScore(), which fits a null model that corrects for gene set-size biases."
- [discussion] Hypothesis-testing framework for candidate annotations: "we demonstrate how to annotate clusters using our Miko Scoring pipeline, which facilitates unbiased scoring of a diverse set of variable-sized cell type-specific gene sets and accepts or rejects"
- [discussion] Limitation of marker-based annotation quality: "we emphasize that marker-based annotations are only as good as the cell-type prescribed to the original dataset."
- [methods] Benchmark dataset files: "so.query <- readRDS("../data/demo/so_tyser2021_220621.rds")"
- [methods] scMiko installation from GitHub: "devtools::install_github(repo = "NMikolajewicz/scMiko")"
