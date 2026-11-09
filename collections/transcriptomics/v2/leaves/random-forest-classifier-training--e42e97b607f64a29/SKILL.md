---
name: random-forest-classifier-training
description: Use when a gene-by-cell scRNA-seq count matrix with trusted per-cell
  labels (cancer, immune, stromal) is available and the goal is to train a hierarchical
  supervised classifier or benchmark it via stratified cross-validation.
when_to_use_negative:
- The input sample is normal (non-cancer) tissue — the default scATOMIC model 'assumes
  the presence of at least some cancer cells in a biopsy and should not be used in
  normal tissue applications'
- Cells are pooled across multiple patients/biopsies — 'Inter-patient cancer cell
  specific effects can interfere with scATOMICs cancer signature scoring module and
  confidence in classifications'
- The task is unsupervised clustering or manual marker-based annotation rather than
  supervised classification against a labeled reference
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scATOMIC
  role: Provides the hierarchical random-forest training scheme (run_scATOMIC layer-wise
    models) and pre-trained pan-cancer TME models
  repo: github.com/abelson-lab/scATOMIC
- name: randomForest
  role: Underlying algorithm for the per-layer classification models
- name: Rmagic
  role: MAGIC imputation of values applied before classification during training/prediction
- name: cutoff.scATOMIC
  role: EM mixture-model-based confidence cutoffs applied to random forest scores
    during training and prediction
  repo: github.com/inofechm/cutoff.scATOMIC
- name: caret
  role: Supports stratified fold creation and performance metric computation
- name: Matrix
  role: Sparse matrix representation of the gene-by-cell count matrix for training
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:f53c72bbe0a8a8929e0980e5e8c65c0c7d3eb5918cadd46a30ef1d0c01d870c4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Matrix
  - Rmagic
  - caret
  - cutoff.scATOMIC
  - randomForest
  - scATOMIC
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# random-forest-classifier-training

## Summary

Train hierarchical random-forest cell-type classifiers (as in scATOMIC) on a pan-cancer single-cell RNA-seq reference (>300,000 cancer, immune, and stromal cells across 19 cancers), and benchmark them with stratified fivefold cross-validation yielding per-cell-type F1 scores.

## When to use

Use when a gene-by-cell scRNA-seq count matrix with trusted per-cell labels (cancer, immune, stromal) is available and the goal is to train a hierarchical supervised classifier or benchmark it via stratified cross-validation.

## When NOT to use

- The input sample is normal (non-cancer) tissue — the default scATOMIC model 'assumes the presence of at least some cancer cells in a biopsy and should not be used in normal tissue applications'
- Cells are pooled across multiple patients/biopsies — 'Inter-patient cancer cell specific effects can interfere with scATOMICs cancer signature scoring module and confidence in classifications'
- The task is unsupervised clustering or manual marker-based annotation rather than supervised classification against a labeled reference

## Inputs

- Sparse gene-by-cell count matrix (Matrix sparseMatrix) from the pan-cancer training reference (>300,000 cells, 19 cancers)
- Per-cell ground-truth labels covering cancer, immune, and stromal populations
- Fold assignment defining five stratified cross-validation folds

## Outputs

- Trained hierarchical random forest models for each layer of the pan-cancer TME hierarchy
- Per-cell predictions for the held-out fold at every hierarchy layer
- Per-cell-type precision, recall, and F1 scores with the median F1 per type

## How to apply

Load the training reference (e.g., the >300,000-cell pan-cancer gene-by-cell count matrix from GSE176078-style data) as a sparse Matrix alongside its cell-type labels. Partition cells into five stratified folds (~61,100 cells each) that preserve cell-type proportions. For each fold, train the layer-wise scATOMIC random forest models on the four training folds with Rmagic (v2.0.3) MAGIC imputation enabled and confidence cutoffs derived with cutoff.scATOMIC (an EM finite-mixture-model cutoff method), then predict the held-out fold at every level of the TME hierarchy. To extend the hierarchy to new subtypes, train additional layers with get_new_scATOMIC_layer() and classify them with classify_new_scATOMIC_layer(). Compare predictions to ground-truth labels per cell type to compute precision, recall, and F1; validated performance spans median F1 of 0.90–0.99 across cell types (Fig. 2a).

## Related tools

- **scATOMIC** (Provides the hierarchical random-forest training scheme (run_scATOMIC layer-wise models) and pre-trained pan-cancer TME models) — github.com/abelson-lab/scATOMIC
- **randomForest** (Underlying algorithm for the per-layer classification models)
- **Rmagic** (MAGIC imputation of values applied before classification during training/prediction)
- **cutoff.scATOMIC** (EM mixture-model-based confidence cutoffs applied to random forest scores during training and prediction) — github.com/inofechm/cutoff.scATOMIC
- **caret** (Supports stratified fold creation and performance metric computation)
- **Matrix** (Sparse matrix representation of the gene-by-cell count matrix for training)

## Examples

```
sparse_matrix <- as(as.matrix(input_matrix), "sparseMatrix"); devtools::install_github("abelson-lab/scATOMIC"); new_layer <- get_new_scATOMIC_layer()
```

## Evaluation signals

- Median per-cell-type F1 across the hierarchy falls within the reported 0.90–0.99 range (Fig. 2a)
- Each of the five folds contains ~61,100 cells and preserves the original cell-type proportions (stratification check)
- Total training reference covers >300,000 cancer, immune, and stromal cells across 19 common cancers
- Per-fold predictions aggregate consistently through all hierarchy layers into a single final label per cell
- Trained models reproduce the run_scATOMIC layer-wise elimination scheme: cells eliminated at a parent node are not scored at child nodes

## Limitations

- The default trained model assumes at least some cancer cells are present and is unsuitable for normal tissue samples
- scATOMIC should be run on individual patient/biopsy samples; inter-patient cancer cell effects can interfere with the cancer signature scoring module and classification confidence
- Datasets >15,000 cells require more than 16 GB RAM; Windows lacks native parallelization and runs slower
- The pre-trained package is large (~minutes to install) because it contains the random forest models; dlm and Rmagic were archived from CRAN and need install_version

## Evidence

- [abstract] We trained scATOMIC on >300,000 cancer, immune, and stromal cells defining a pan-cancer reference across 19 common cancers: "We trained scATOMIC on >300,000 cancer, immune, and stromal cells defining a pan-cancer reference across 19 common cancers and employ a hierarchical approach"
- [intro] hierarchical models reduce transcriptomic complexity: "We devise a structured scheme that uses hierarchically organized models and elimination processes, reducing the transcriptomic complexity of the TME multi-cellular system"
- [methods] train new subclassification layers: "we run the get_new_scATOMIC_layer() function"
- [methods] MAGIC imputation used by scATOMIC: "In scATOMIC we use the Rmagic package to impute values."
- [readme] cutoff.scATOMIC EM mixture-model method: [evidence span withheld — anchor status: external_doc]
- [discussion] outperformance on training reference: "scATOMIC is comparable to or outperforms other existing automatic cell type annotators when classifying blood and stromal cells using our training reference."
