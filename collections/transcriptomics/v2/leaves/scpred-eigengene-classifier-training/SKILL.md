---
name: scpred-eigengene-classifier-training
description: Use when a benchmark dataset has known ground-truth cell-type labels
  and per-cell supervised annotation accuracy must be quantified against other tools
  (e.g., EasyCellType, Garnett, SingleR).
when_to_use_negative:
- No labeled reference cells are available — marker-list querying methods like EasyCellType
  (GSEA / modified Fisher's exact test) are the appropriate choice instead
- Input is only a differential-expression marker list to check against cell marker
  databases — that is EasyCellType's workflow, not scPred training
- The task is cluster-level rather than per-cell annotation accuracy — supervised
  per-cell classifiers will not directly produce that output
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scPred
  role: Supervised eigengene-based classifier trained on 3500 sampled cells per dataset;
    run in prediction and no-rejection variants
- name: Seurat
  role: Standard preprocessing pipeline (normalization, variable feature selection,
    scaling, PCA, clustering) applied to simulated PBMC datasets before annotation
- name: EasyCellType
  role: Marker-based annotation baseline that scPred's supervised accuracy is benchmarked
    against in the Supplementary Figure S5 comparison
  repo: https://github.com/rx-li/EasyCellType
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:21adcbf531e22f35a43e4597ff0b6dac225159ec61ee4c7c61bf107822771972
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - EasyCellType
  - Seurat
  - scPred
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# scpred-eigengene-classifier-training

## Summary

Supervised cell-type annotation skill that trains scPred's eigengene-based machine-learning classifier on a labeled reference subset (3500 sampled cells per simulated PBMC dataset) and scores per-cell prediction accuracy in both prediction and no-rejection variants.

## When to use

Use when a benchmark dataset has known ground-truth cell-type labels and per-cell supervised annotation accuracy must be quantified against other tools (e.g., EasyCellType, Garnett, SingleR).

## When NOT to use

- No labeled reference cells are available — marker-list querying methods like EasyCellType (GSEA / modified Fisher's exact test) are the appropriate choice instead
- Input is only a differential-expression marker list to check against cell marker databases — that is EasyCellType's workflow, not scPred training
- The task is cluster-level rather than per-cell annotation accuracy — supervised per-cell classifiers will not directly produce that output

## Inputs

- Simulated (or real) PBMC scRNA-seq count matrix with known ground-truth cell-type labels
- Seurat-processed dataset (normalized, scaled, PCA-reduced, clustered)
- Labeled training subset of 3500 sampled cells per dataset

## Outputs

- Per-cell predicted cell-type labels in prediction and no-rejection variants
- Per-cell annotation accuracy values per tool/variant and dataset size, combined into figure_s5_accuracy_table.csv

## How to apply

Preprocess each simulated PBMC count matrix with the standard Seurat pipeline (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors, FindClusters) so that cells and their ground-truth labels sit in a consistent representation. Sample 3500 cells per dataset as the labeled training set and train scPred, which derives discriminant features from principal-component eigengene coordinates of the expression matrix and fits a supervised classifier against the known cell-type labels. Generate predictions for the remaining cells in two variants — the default mode, which may reject uncertain cells, and the no-rejection variant, which forces a label on every cell — so rejection behavior does not confound the comparison. Compute per-cell annotation accuracy against the simulated ground-truth labels for each dataset size and variant, and write the combined table (e.g., figure_s5_accuracy_table.csv) for side-by-side benchmarking against EasyCellType's soft/hard classifiers, Garnett, and SingleR.

## Related tools

- **scPred** (Supervised eigengene-based classifier trained on 3500 sampled cells per dataset; run in prediction and no-rejection variants)
- **Seurat** (Standard preprocessing pipeline (normalization, variable feature selection, scaling, PCA, clustering) applied to simulated PBMC datasets before annotation)
- **EasyCellType** (Marker-based annotation baseline that scPred's supervised accuracy is benchmarked against in the Supplementary Figure S5 comparison) — https://github.com/rx-li/EasyCellType

## Evaluation signals

- Training set size equals 3500 sampled cells per simulated dataset as specified in the benchmark protocol
- Predictions are generated in both prediction and no-rejection variants; no-rejection accuracy should be at least as high as the default variant since no cells are left unassigned
- Per-cell accuracy values are computed against the simulated ground-truth labels for every tool/variant and dataset size and are internally consistent across replicate datasets
- Qualitative pattern reproduces Supplementary Figure S5: supervised methods generally outperform EasyCellType's hard classifiers
- Combined accuracy table written to figure_s5_accuracy_table.csv contains one row per tool/variant/dataset-size combination

## Limitations

- Supervised training requires ground-truth labels; the article notes simulation and real-data applications were needed to assess performance, and supervised tools cannot annotate unlabeled marker lists directly
- Comparisons are confounded by rejection behavior, so the no-rejection variant must be reported alongside the default prediction mode
- Version numbers of the annotation tools are not recorded in the article's methods, limiting exact reproducibility

## Evidence

- [supplementary] Train scPred on 3500 sampled cells per dataset and generate predictions in both prediction and no-rejection variants.: [evidence span withheld — anchor status: not_found]
- [supplementary] Accuracy table output from the benchmark: [evidence span withheld — anchor status: not_found]
- [methods] Seurat preprocessing of PBMC data: "We followed the standard workflow provided by `Seurat` package[@seurat] to process the PBMC data set."
- [abstract] Simulation-based favorable results for EasyCellType: "The simulation study and real-data applications demonstrate favorable results by the proposed method."
