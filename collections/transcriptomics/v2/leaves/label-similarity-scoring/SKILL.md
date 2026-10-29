---
name: label-similarity-scoring
description: Use when benchmarking cell-type annotation methods (e.g., Census vs scType)
  on datasets with harmonized ground-truth labels and you need a per-dataset label-similarity
  score that captures partial agreement between label vocabularies, not just exact-match
  accuracy.
when_to_use_negative:
- Reference and predicted annotations use identical flat label strings so exact-match
  accuracy/F1 alone fully captures agreement and a graded similarity is unnecessary
- The datasets lack harmonized ground-truth labels (e.g., purely exploratory annotation
  of new data), leaving no reference to score similarity against
- Only one annotation method is being applied and no comparative benchmarking across
  datasets is intended
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Census
  role: Produces the per-cell predictions being scored; hierarchical gradient-boosted
    decision tree annotation with UMAP-space label stabilization
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: Loads and preprocesses each tissue dataset (NormalizeData through RunUMAP)
    into the Seurat object format Census requires before prediction and scoring
- name: ggpubr
  role: Runs Wilcoxon signed-rank statistical comparison of the label-similarity/F1/accuracy
    distributions between Census and scType across the 44 datasets
  repo: kassambara/ggpubr
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:9a7b9b99953a23dee448fbbe6297d7f35251591cae6af7e8b56e3ddf0b8c2c4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat 4
  - ggpubr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# label-similarity-scoring

## Summary

Computes a label similarity metric that quantifies how close predicted cell-type annotations (e.g., from Census hierarchical gradient-boosted decision trees) are to harmonized reference labels, alongside F1 and balanced accuracy, when benchmarking annotation methods across the 44 held-out human and mouse tissue datasets.

## When to use

Use when benchmarking cell-type annotation methods (e.g., Census vs scType) on datasets with harmonized ground-truth labels and you need a per-dataset label-similarity score that captures partial agreement between label vocabularies, not just exact-match accuracy.

## When NOT to use

- Reference and predicted annotations use identical flat label strings so exact-match accuracy/F1 alone fully captures agreement and a graded similarity is unnecessary
- The datasets lack harmonized ground-truth labels (e.g., purely exploratory annotation of new data), leaving no reference to score similarity against
- Only one annotation method is being applied and no comparative benchmarking across datasets is intended

## Inputs

- Seurat object (Seurat v4) with UMAP dimensionality reduction, per-cell reference (harmonized) labels, and normalized counts
- Per-cell predicted annotations from Census (census_main output, res$pred) and from scType
- Harmonized label vocabulary covering the 105 reference subtypes

## Outputs

- Per-dataset label similarity score between predicted and reference annotations
- Companion per-dataset F1, balanced accuracy, and overall accuracy values
- Aggregated metric table (e.g., benchmark_metrics.csv) for Wilcoxon signed-rank comparison across the 44 datasets

## How to apply

Load each of the held-out atlas-scale normal/cancer tissue datasets (1,769,071 cells, 105 harmonized labels) into Seurat v4, process through the standard pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP dims=1:50), and generate per-cell predictions with census_main for Census and with scType. For each dataset, compare predicted labels against the harmonized reference labels by computing label similarity (a graded score tolerating related labels rather than requiring exact matches) together with F1 and balanced accuracy. Collect per-dataset metrics and evaluate differences between methods with Wilcoxon signed-rank tests across the 44 datasets (ggpubr), accepting significance at p < 0.05. Rationale: because Census predicts hierarchical subtypes that may not match reference label strings exactly, label similarity complements strict F1/accuracy by rewarding predictions that are consistent with the reference label ontology.

## Related tools

- **Census** (Produces the per-cell predictions being scored; hierarchical gradient-boosted decision tree annotation with UMAP-space label stabilization) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Loads and preprocesses each tissue dataset (NormalizeData through RunUMAP) into the Seurat object format Census requires before prediction and scoring)
- **ggpubr** (Runs Wilcoxon signed-rank statistical comparison of the label-similarity/F1/accuracy distributions between Census and scType across the 44 datasets) — kassambara/ggpubr

## Evaluation signals

- Wilcoxon signed-rank test across the 44 datasets shows Census label similarity significantly higher than scType (p < 0.05; reported Wilcoxon P < 2e−16)
- Per-dataset similarity values fall in a consistent range with the companion F1 and balanced accuracy metrics, without contradictions (e.g., high similarity but zero exact-match F1 should trigger label-vocabulary mapping review)
- All 1,769,071 cells across 105 harmonized labels receive both a prediction and a similarity score; no cells dropped from the comparison
- Census prediction counts are internally consistent with the reported subtype tally (81/105 subtypes identified on the held-out benchmark)

## Limitations

- Label similarity depends on the harmonized label set (105 labels); non-harmonized or novel subtypes cannot be fairly scored, and Census correctly identified 81/105 tested subtypes while scType identified only 35/89, so coverage differences complicate direct metric reading
- Census requires Seurat version 4 to run, and the main model covers 175 cell-types from 24 organs plus a limited set of cancer models (Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas); tissues outside this scope cannot be annotated or scored with the pretrained model
- scType achieved shorter run times than Census (~13,000 cells/min for Census), so throughput trade-offs accompany the higher label-similarity scores

## Evidence

- [abstract] Benchmark on 44 tissues showing Census outperforms state-of-the-art across multiple metrics: "When benchmarked on 44 atlas-scale normal and cancer, human and mouse tissues, Census significantly outperforms state-of-the-art methods across multiple metrics"
- [intro] Census annotation and label-stabilizing mechanism that underlies the predictions being scored: "New datasets are annotated using the pretrained models followed by a custom-developed label-stabilizing algorithm (Fig. 1b). This involves using clusters and prediction contours in UMAP space"
- [readme] Census definition and scope for scRNA-seq annotation: "Census is a fast and fully automated hierarchical cell-type identification method in R for single-cell RNA-seq (scRNA-seq)."
- [readme] Seurat v4 dependency required before scoring predictions: "Note: a demo script (demo.r) and dataset (peng_sub.RDS) are also included in this directory. Also note: Census requires Seurat version 4 to run."
- [readme] Standard Seurat preprocessing pipeline that must precede census_main prediction: "obj = obj %>% CreateSeuratObject() %>% NormalizeData() %>% ScaleData() %>%
  FindVariableFeatures() %>% RunPCA() %>% RunUMAP(dims=1:50)"
