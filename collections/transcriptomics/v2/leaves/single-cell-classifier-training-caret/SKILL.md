---
name: single-cell-classifier-training-caret
description: Use when a single-cell (RNA or CyTOF) expression matrix needs a probabilistic
  classifier for active learning uncertainty sampling, pseudo-labelling, or entropy-based
  mislabel detection.
when_to_use_negative:
- When a dedicated single-cell annotation method is required instead of a general-purpose
  classifier (e.g. SingleR, scmap, SingleCellNet are implemented with default parameters
  for direct comparison)
- When only raw counts are available without normalization — the models require logcounts
  scaled/centered and reduced to the first 20 PCs
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: caret
  role: R package used to train the random forest and logistic regression classifiers
    on the first 20 PCs of scaled and centered logcounts
  repo: https://github.com/camlab-bioml/leader
- name: scuttle
  role: Computes logcounts and size-factor normalization prior to PCA and classifier
    training
- name: leader (R package)
  role: Orchestrates active learning and adaptive reweighting cell selection that
    uses these classifiers' uncertainty estimates
  repo: https://github.com/camlab-bioml/leader
- name: scran
  role: Provides findMarkers for marker-aware analyses in the same benchmarking pipeline
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:3cde67752ec662155b471ac52236fb24d2256c626bfe1d2a17c49b645739ee0b
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 21
---

# single-cell-classifier-training-caret

## Summary

Train random forest and logistic regression classifiers on the first 20 principal components of scaled and centered logcount expression data using the caret R package, as the surrogate annotation models used for active learning, self-training and mislabel detection in the Geuenich et al. 2024 single-cell benchmarking study.

## When to use

Use when a single-cell (RNA or CyTOF) expression matrix needs a probabilistic classifier for active learning uncertainty sampling, pseudo-labelling, or entropy-based mislabel detection.

## When NOT to use

- When a dedicated single-cell annotation method is required instead of a general-purpose classifier (e.g. SingleR, scmap, SingleCellNet are implemented with default parameters for direct comparison)
- When only raw counts are available without normalization — the models require logcounts scaled/centered and reduced to the first 20 PCs

## Inputs

- Processed single-cell expression matrix (Seurat object or SingleCellExperiment, e.g. from sc_mixology SCE objects) with cell type labels
- Logcounts with size factors computed via scuttle
- Cell-level metadata with ground truth labels (e.g. cell_line_demuxlet column for sc_mixology)
- Active-learning-selected training sets of 100, 250 or 500 labeled cells per 50:50 stratified train/test split

## Outputs

- Trained random forest and logistic regression caret model objects
- Class posterior probabilities per cell for entropy computation
- Predicted labels with confidence rankings (top 10%, 50%, 100% most confidently predicted cells for pseudo-labeling)
- Scaled predictive entropy values per cell (mislabeled vs correctly labeled cells)

## How to apply

First compute logcounts and size-factor normalization with the scuttle R package, then scale and center the logcount expression matrix. Compute the first 20 principal components and use them as the input feature matrix. Train a random forest and a logistic regression classifier with the caret R package, and use the class posterior probabilities to derive predictive entropy (scaled for comparability across cohorts) for active learning cell selection and self-training pseudo-label selection. These classifiers are retrained iteratively in the active learning loop (selecting the 10 most uncertain cells each round) and on the combined ground-truth plus self-labeled training set for self-training experiments.

## Related tools

- **caret** (R package used to train the random forest and logistic regression classifiers on the first 20 PCs of scaled and centered logcounts) — https://github.com/camlab-bioml/leader
- **scuttle** (Computes logcounts and size-factor normalization prior to PCA and classifier training)
- **leader (R package)** (Orchestrates active learning and adaptive reweighting cell selection that uses these classifiers' uncertainty estimates) — https://github.com/camlab-bioml/leader
- **scran** (Provides findMarkers for marker-aware analyses in the same benchmarking pipeline)

## Examples

```
seu <- NormalizeData(seu); seu <- FindVariableFeatures(seu, selection.method='vst', nfeatures=2000); all.genes <- rownames(seu); seu <- ScaleData(seu, features=all.genes); rf_fit <- train(x = pcs[,1:20], y = labels, method = 'rf', trControl = ctrl)
```

## Evaluation signals

- Predictive entropy of 10% artificially corrupted-label cells should be clearly higher than that of correctly labeled cells in each cohort and train split (Fig. 5D comparison)
- Self-training on confidently predicted cells (top 10-50%) should yield positive F1 improvement for the most effective annotation methods (Fig. 5B direction and ordering)
- The classifier input matrix must have exactly 20 features (the first 20 PCs of scaled, centered logcounts)
- Balanced accuracy and sensitivity on held-out test cells should reproduce the relative ordering of active learning vs random selection in Fig. 3B rank tables

## Limitations

- Ground truth labels were derived from previous studies rather than manual annotation, so label quality is inherently ambiguous
- Reproducibility gaps acknowledged: no sample size predetermination, no randomization, no investigator blinding
- Optimal certainty threshold for self-training cell selection was not identified and is left to future work

## Evidence

- [discussion] caret training on PCs: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package54"
- [discussion] self-training selection by entropy: "We then selected the top 10, 50 and 100% most confidently predicted cells based on their entropy and labeled these using the prediction values obtained."
- [discussion] normalization with scuttle: "we calculated logcounts and normalized with the size factor using the scuttle R package52"
- [discussion] active learning loop step size: "we selected the set of 10 cells with the highest uncertainty to label using ground truth data. After this set of cells is labeled, the classifier is trained again and the loop repeats."
- [abstract] leader package availability: "An R package implementing the heuristic approaches introduced in this work may be found at https://github.com/camlab-bioml/leader"
