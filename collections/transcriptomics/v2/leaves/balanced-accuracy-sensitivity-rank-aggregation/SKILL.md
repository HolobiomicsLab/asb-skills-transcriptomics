---
name: balanced-accuracy-sensitivity-rank-aggregation
description: Use when comparing multiple cell/training-set selection strategies evaluated
  across several datasets and classifiers, and you need a single robust overall ranking
  from per-combination balanced accuracy and sensitivity results.
when_to_use_negative:
- Input is a single dataset or single classifier — per-combination accuracy comparison
  is sufficient and rank aggregation adds no information
- The evaluation goal is per-cell-type performance or confusion structure rather than
  overall method ranking — balanced accuracy medians hide class-specific failure modes
- Cell selection strategies are being compared on imbalanced synthetic datasets (e.g.
  450 vs 50 cells) where the paper's dedicated imbalance experiments, not Fig. 3B
  aggregation, are the appropriate analysis
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: leader
  role: R package implementing adaptive reweighting (cluster-based and marker_based
    modes) used to generate the training sets being ranked
  repo: https://github.com/camlab-bioml/leader
- name: SingleR
  role: Cell type annotation classifier trained on each selection-method training
    set and evaluated in the ranking
- name: scmap
  role: Cell type annotation classifier evaluated in the benchmarking ranks
- name: SingleCellNet
  role: Cell type annotation classifier implemented with default parameters for evaluation
- name: caret
  role: Trains the random forest (on first 20 PCs of scaled, centered logcounts) and
    logistic regression classifiers used in evaluation
- name: Seurat
  role: Clustering step underlying cluster-based adaptive reweighting cell selection
- name: scuttle
  role: Computes logcounts and size-factor normalization of expression matrices prior
    to classifier training
- name: scran
  role: findMarkers differential expression used to derive cell-line markers for the
    marker-informed variant
- name: HDCytoData
  role: Source of the CyTOF bone marrow dataset (24 cell types, 10 mice) included
    in the cross-dataset ranking
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:c76a2c9534e120103d23626a09f3773f4bdde6217ad2f80c28a2972d72d7b219
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - HDCytoData
  - Seurat
  - SingleCellNet
  - SingleR
  - caret
  - leader
  - leader (R package)
  - scmap
  - scran
  - scuttle
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# balanced-accuracy-sensitivity-rank-aggregation

## Summary

Aggregate benchmark results into median balanced accuracy and sensitivity ranks per cell selection method (active learning, adaptive reweighting, random) across datasets and classifiers to identify which selection strategy performs best overall, as done for Fig. 3B of the single-cell annotation benchmarking study.

## When to use

Use when comparing multiple cell/training-set selection strategies evaluated across several datasets and classifiers, and you need a single robust overall ranking from per-combination balanced accuracy and sensitivity results.

## When NOT to use

- Input is a single dataset or single classifier — per-combination accuracy comparison is sufficient and rank aggregation adds no information
- The evaluation goal is per-cell-type performance or confusion structure rather than overall method ranking — balanced accuracy medians hide class-specific failure modes
- Cell selection strategies are being compared on imbalanced synthetic datasets (e.g. 450 vs 50 cells) where the paper's dedicated imbalance experiments, not Fig. 3B aggregation, are the appropriate analysis

## Inputs

- Balanced accuracy and sensitivity values for every dataset x classifier x selection-method combination over ten stratified 50:50 train/test splits
- Training sets of 100, 250, and 500 labeled cells produced by each selection method (active learning, cluster-based adaptive reweighting, marker-informed adaptive reweighting, random)
- Six benchmark datasets: breast cancer cell line atlas, sc_mixology lung cancer cell lines, SCP1089 untreated PDAC snRNA-Seq (6,000-cell subsample), HDCytoData CyTOF bone marrow, Liver Cell Atlas, Tabula Sapiens release 1.0

## Outputs

- Median balanced accuracy rank per selection method aggregated across datasets and classifiers
- Median sensitivity rank per selection method aggregated across datasets and classifiers
- Aggregated rank table (e.g. fig3b_selection_method_ranks.csv) reproducing the Fig. 3B pattern

## How to apply

For every dataset x classifier x selection-method combination, evaluate balanced accuracy and sensitivity on held-out test cells (here: ten 50:50 train/test splits stratified by cell type, training sets of 100, 250, and 500 labeled cells). Compare selection methods by ranking their balanced accuracy (and separately sensitivity) against each other within each dataset-classifier pair, then compute the median rank per method across all datasets and classifiers. Methods benchmarked this way were maximum-entropy and lowest-max-probability active learning (10 most uncertain cells selected per iteration), cluster-based adaptive reweighting (Seurat clustering with equal cells per cluster), marker-informed adaptive reweighting (leader package, mode='marker_based'), and random selection. Write the aggregated ranks to a table (e.g. fig3b_selection_method_ranks.csv) and verify it reproduces the expected pattern. Rationale: median ranks across datasets and classifiers are more robust than raw accuracy values to scale differences between datasets and classifier families (SingleR, scmap, SingleCellNet, CyTOF-LDA, random forest, SVM).

## Related tools

- **leader** (R package implementing adaptive reweighting (cluster-based and marker_based modes) used to generate the training sets being ranked) — https://github.com/camlab-bioml/leader
- **SingleR** (Cell type annotation classifier trained on each selection-method training set and evaluated in the ranking)
- **scmap** (Cell type annotation classifier evaluated in the benchmarking ranks)
- **SingleCellNet** (Cell type annotation classifier implemented with default parameters for evaluation)
- **caret** (Trains the random forest (on first 20 PCs of scaled, centered logcounts) and logistic regression classifiers used in evaluation)
- **Seurat** (Clustering step underlying cluster-based adaptive reweighting cell selection)
- **scuttle** (Computes logcounts and size-factor normalization of expression matrices prior to classifier training)
- **scran** (findMarkers differential expression used to derive cell-line markers for the marker-informed variant)
- **HDCytoData** (Source of the CyTOF bone marrow dataset (24 cell types, 10 mice) included in the cross-dataset ranking)

## Examples

```
markers <- get_markers(system.file("extdata", "markers.yml", package = "leader")); adaptive_reweighting(seu, markers, n_requested_cells = 500, mode = "marker_based")
```

## Evaluation signals

- The aggregated median-rank table (fig3b_selection_method_ranks.csv) reproduces the Fig. 3B pattern: active learning outranks random selection and adaptive reweighting across most datasets, with adaptive reweighting competitive in some settings
- No single method is expected to be consistently best on every dataset — a table showing one method winning universally indicates an aggregation or split error
- Each of the ten stratified 50:50 splits and all three training-set sizes (100, 250, 500 cells) are represented for every dataset x classifier x method combination before ranking
- Balanced accuracy and sensitivity are ranked separately; sensitivity ranks should be consistent with balanced accuracy ranks within expected benchmark variability

## Limitations

- 'Ground truth' labels were taken from previous studies rather than manual annotation, introducing ambiguity in what the ranking actually measures
- Marker definitions are circular for cell line data since markers derive from differential expression between the cell lines being classified
- Benchmarking favored breadth of annotation tools and hyperparameters over number of datasets, so aggregated ranks rest on six datasets
- No statistical method was used to predetermine sample size and experiments were not randomized or blinded

## Evidence

- [results] no single cell selection method is consistently best; however, active learning outperforms random selection and adaptive reweighting across most datasets: [evidence span withheld — anchor status: not_found]
- [methods] 50:50 stratified splits evidence: "Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type"
- [discussion] adaptive reweighting procedure: "To accomplish this, we cluster with Seurat13. Next, we sampled cells from the resulting clusters such that the same number of cells is taken from each cluster."
- [discussion] classifier training evidence: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package54"
- [readme] leader README usage: "adaptive_reweighting(seu, markers, n_requested_cells = 40, mode = "marker_based")"
