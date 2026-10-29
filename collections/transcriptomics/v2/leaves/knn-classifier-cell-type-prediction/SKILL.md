---
name: knn-classifier-cell-type-prediction
description: Use when a query scRNA-seq dataset has been mapped into a Symphony reference
  embedding with mapQuery and you need cell type labels for query cells without retraining
  from raw reference data.
when_to_use_negative:
- When the query may contain cell types absent from the reference — Symphony does
  not replace de novo analyses for novel cell type discovery, and mapping metrics
  are less informative when a novel population resembles an existing reference population
- When data has not been mapped into the harmonized reference embedding — kNN voting
  on unmapped coordinates or a different feature space invalidates the annotation
  transfer
- When you need de novo clustering/integration of the query alone rather than transfer
  of reference-defined annotations
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Symphony
  role: Provides buildReference and mapQuery to place query cells in the harmonized
    embedding on which knnPredict performs k-nearest-neighbor cell type voting
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Performs the donor/protocol integration underlying the reference embedding
    before compression and kNN classification
  repo: https://github.com/immunogenomics/symphony
- name: scmap
  role: Benchmarked automated cell type classifier against which Symphony kNN predictions
    are compared in the PbmcBench evaluation
  repo: https://github.com/tabdelaal/scRNAseq_Benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:6b90217b700db68b984d6cd4024eb20694c5dc2b4ccbaff753f60cfd45e91ef5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  - scmap
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# knn-classifier-cell-type-prediction

## Summary

Predict query cell types via k-nearest-neighbor voting (k=5) in the Symphony harmonized reference embedding, transferring reference-defined annotations and yielding confidence scores. This is the Symphony-recommended two-step annotation transfer: learn the classifier in the reference embedding, then apply it to mapped query coordinates.

## When to use

Use when a query scRNA-seq dataset has been mapped into a Symphony reference embedding with mapQuery and you need cell type labels for query cells without retraining from raw reference data.

## When NOT to use

- When the query may contain cell types absent from the reference — Symphony does not replace de novo analyses for novel cell type discovery, and mapping metrics are less informative when a novel population resembles an existing reference population
- When data has not been mapped into the harmonized reference embedding — kNN voting on unmapped coordinates or a different feature space invalidates the annotation transfer
- When you need de novo clustering/integration of the query alone rather than transfer of reference-defined annotations

## Inputs

- Symphony reference object (RDS) with harmonized embedding, cell metadata, and cell type labels
- Query expression matrix normalized as log(CP10k+1) with matching gene features
- Reference mapQuery output containing query cells' coordinates in the reference embedding
- uwot model saved per reference split for query projection

## Outputs

- Per-query-cell predicted cell type labels (majority vote of k=5 reference neighbors)
- Per-cell kNN confidence scores based on neighbor vote fractions
- Optionally, per-cell-type F1 scores and median F1 per experiment (e.g. f1_benchmark_results.csv)

## How to apply

First build a Symphony reference (vst variable gene selection of top 1,000 genes per group, 20 PCs, K=100 Harmony clusters, donor integration) and normalize the query as log(CP10k+1) before mapQuery places it in the harmonized embedding. Then train a k-nearest-neighbor classifier on the reference cells' embedding coordinates and labels, and call knnPredict with k=5 so each query cell votes across its 5 nearest reference neighbors. The classifier returns predicted cell type labels plus per-cell confidence scores derived from the neighbor votes. Score per-cell-type F1 against held-out labels and take the median across cell types (e.g. with the Abdelaal et al. evaluate utility) to judge performance; in the PbmcBench cross-technology benchmark, Symphony kNN was one of three downstream classifiers (alongside linear SVM and multinomial logistic regression) achieving average median F1 of 0.79-0.87 across 48 train-test experiments.

## Related tools

- **Symphony** (Provides buildReference and mapQuery to place query cells in the harmonized embedding on which knnPredict performs k-nearest-neighbor cell type voting) — https://github.com/immunogenomics/symphony
- **Harmony** (Performs the donor/protocol integration underlying the reference embedding before compression and kNN classification) — https://github.com/immunogenomics/symphony
- **scmap** (Benchmarked automated cell type classifier against which Symphony kNN predictions are compared in the PbmcBench evaluation) — https://github.com/tabdelaal/scRNAseq_Benchmark

## Examples

```
query = mapQuery(query_exp, query_meta, reference, vars = c('donor')); query = knnPredict(query, reference, reference$meta_data$cell_type, k = 5)
```

## Evaluation signals

- Average median per-cell-type F1 across the 48 PbmcBench train-test experiments falls in the reported 0.79-0.87 range, comparable to top supervised classifiers (scmap-cell, singleCellNet, SCINA) at 0.77-0.83
- Per-experiment F1 table (f1_benchmark_results.csv) contains a median F1 per cell type for every train-test split
- Every query cell receives a label and a confidence score, with confidence reflecting vote fractions among the k=5 nearest reference neighbors
- Query cells localize within the same low-dimensional reference embedding as integrated reference cells (verifiable by mixing metrics such as LISI)
- Query normalization matches reference preprocessing (log(CP10k+1)) and variable gene set (top 1,000 vst genes per group)

## Limitations

- Symphony mapping metrics may be less informative when a novel population is very similar to an existing reference population
- Identification of novel cell types that have failed to map is an unsolved problem; kNN will force query cells onto existing reference labels
- Condition I requiring comprehensive cell type coverage in the reference is inflexible and limits equivalence with de novo Harmony integration
- Transfer of annotations from reference to query remains an open area of research, so kNN voting is one of several possible downstream classifiers (linear SVM, multinomial logistic regression also apply)

## Evidence

- [abstract] Symphony localizes query cells within a stable low-dimensional reference embedding, facilitating transfer of reference-defined annotations to the query: [evidence span withheld — anchor status: not_found]
- [methods] knnPredict on query object: "query = knnPredict(query,       # query object"
- [discussion] Two-step annotation transfer: "We first learn a predictive model in the reference embedding, and then map query cells and use their reference coordinates to predict query annotations."
- [intro] Automated classifiers assign rigid supervised annotations: "This is in contrast to automated cell type classifiers, such as scmap31, which assign rigid annotations based on reference datasets in a supervised manner"
