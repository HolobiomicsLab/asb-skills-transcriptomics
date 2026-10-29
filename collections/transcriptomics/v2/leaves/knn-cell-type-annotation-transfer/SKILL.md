---
name: knn-cell-type-annotation-transfer
description: Use when a query scRNA-seq dataset has been mapped into a Symphony reference
  embedding and you need reference-defined cell type labels with per-cell confidence,
  plus accuracy/median-F1 validation against held-out true labels.
when_to_use_negative:
- When the query contains novel cell types absent from the reference — Symphony requires
  comprehensive cell type coverage in the reference (Condition I), and mapping metrics
  are less informative for unmapped novel populations
- When the goal is de novo discovery of novel cell states in the query rather than
  transfer of reference-defined annotations
- When a novel query population is very similar to an existing reference population
  — prediction metrics and mapping confidence may be uninformative even when mapping
  is imperfect
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Symphony
  role: Provides knnPredict for k-NN voting annotation transfer (e.g., k = 5 or k
    = 30 with confidence = TRUE) on the mapped query object
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Integrates reference donors/technologies upstream so that query cells are
    placed in a harmonized embedding in which k-NN label transfer is performed
  repo: https://github.com/immunogenomics/symphony
- name: tabdelaal/scRNAseq_Benchmark
  role: Source of the PbmcBench experiments and benchmarking context against supervised
    classifiers (scmap-cell, SingleCellNet, SCINA) whose average median F1 (0.77-0.83)
    Symphony-based classifiers match (0.79-0.87)
  repo: https://github.com/tabdelaal/scRNAseq_Benchmark
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:5648424707c31aebdf208789269c5102b332f17a1551a84b5c5073c666bbebda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Seurat
  - Symphony
  - dplyr
  - irlba
  - scArches
  - tabdelaal/scRNAseq_Benchmark
  - uwot
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# knn-cell-type-annotation-transfer

## Summary

Transfer reference cell-type annotations to mapped query cells using k-nearest-neighbor voting (Symphony's knnPredict) in the harmonized reference embedding, with per-cell confidence scores, and score the transfer with accuracy and per-class F1 via the evaluate utility.

## When to use

Use when a query scRNA-seq dataset has been mapped into a Symphony reference embedding and you need reference-defined cell type labels with per-cell confidence, plus accuracy/median-F1 validation against held-out true labels.

## When NOT to use

- When the query contains novel cell types absent from the reference — Symphony requires comprehensive cell type coverage in the reference (Condition I), and mapping metrics are less informative for unmapped novel populations
- When the goal is de novo discovery of novel cell states in the query rather than transfer of reference-defined annotations
- When a novel query population is very similar to an existing reference population — prediction metrics and mapping confidence may be uninformative even when mapping is imperfect

## Inputs

- Symphony query object produced by mapQuery (query cells projected into the reference harmonized embedding, optionally with UMAP via do_umap = TRUE)
- Symphony reference object (e.g., from buildReference or buildReferenceFromHarmonyObj) containing reference cell-type labels
- Query expression matrix normalized to match the reference convention (log(CP10k+1)), scaled as required
- Held-out true cell type labels for the query (for F1/accuracy evaluation)

## Outputs

- Per-cell predicted cell type labels for the query
- Per-cell annotation confidence scores (with confidence = TRUE)
- Per-class F1 scores, median F1 across cell types, and overall accuracy (via evaluate)
- Tables such as f1_benchmark_results.csv or query_mapping_results.csv with predictions, confidences, and metrics

## How to apply

After running mapQuery to place query cells in the reference harmonized embedding, call Symphony's knnPredict on the query object using the reference cell-type labels and set confidence = TRUE to obtain per-cell prediction confidence scores. Choose k per experiment: k = 5 neighbors is used in the cross-technology PBMC leave-one-technology-out folds and the pancreatic islet (10,455-cell inDrop human+mouse query) mapping, while k = 30 is used for the fetal liver hematopoiesis query. Score predictions against held-out true labels using the evaluate function, computing overall accuracy, per-class F1, and the median F1 across cell types; on PbmcBench, repeat across the 48 protocol/donor train-test partitions and average the median F1. The rationale is Symphony's two-step annotation transfer: learn a predictive model in the reference embedding, then use each query cell's reference coordinates to predict its annotation, which reproduces the stable low-dimensional localization the method was designed for.

## Related tools

- **Symphony** (Provides knnPredict for k-NN voting annotation transfer (e.g., k = 5 or k = 30 with confidence = TRUE) on the mapped query object) — https://github.com/immunogenomics/symphony
- **Harmony** (Integrates reference donors/technologies upstream so that query cells are placed in a harmonized embedding in which k-NN label transfer is performed) — https://github.com/immunogenomics/symphony
- **tabdelaal/scRNAseq_Benchmark** (Source of the PbmcBench experiments and benchmarking context against supervised classifiers (scmap-cell, SingleCellNet, SCINA) whose average median F1 (0.77-0.83) Symphony-based classifiers match (0.79-0.87)) — https://github.com/tabdelaal/scRNAseq_Benchmark

## Examples

```
query <- knnPredict(query, reference, query$exp_scaled, k = 5, confidence = TRUE)
```

## Evaluation signals

- Cross-technology PBMC leave-one-technology-out folds: 5-NN (k = 5) classification reaches ~97.5% accuracy with median F1 ~0.97
- Fetal liver 10x 5' query at k = 30: median cell type F1 reaches 0.83 and overall accuracy reaches 85.0%
- PbmcBench 48 train-test experiments: average median cell type F1 falls in the reported 0.79-0.87 range, comparable to scmap-cell/SingleCellNet/SCINA (0.77-0.83)
- Pancreatic islet mapping at k = 5: human cell-type F1 ~0.96 and mouse F1 ~0.95, with Symphony outperforming Seurat and scArches
- knnPredict with confidence = TRUE returns per-cell confidence scores alongside predictions; verify both are populated in the output CSV

## Limitations

- Condition I (comprehensive cell type coverage in the reference) is not flexible and limits equivalence with de novo Harmony integration
- Identification of novel cell types that have failed to map is an unsolved future direction for mapping algorithms
- Symphony mapping metrics may be less informative when a novel population is very similar to an existing reference population

## Evidence

- [discussion] Two-step annotation transfer: learn a predictive model in the reference embedding, then map query cells and use their reference coordinates to predict annotations: "We first learn a predictive model in the reference embedding, and then map query cells and use their reference coordinates to predict query annotations."
- [abstract] Symphony localizes query cells within a stable low-dimensional reference embedding, enabling reproducible annotation transfer: "Symphony localizes query cells within a stable low-dimensional reference embedding, facilitating reproducible downstream transfer of reference-defined annotations"
- [methods] knnPredict applied to the query object in the methods workflow: "query = knnPredict(query,       # query object"
- [discussion] Reference compression enables fast, seconds-scale query mapping that underpins the k-NN label transfer: "Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds."
