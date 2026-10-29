---
name: single-cell-batch-effect-embedding-training
description: Use when single-cell RNA-seq data from multiple batches requires a nonlinear
  lower-dimensional embedding that explicitly models batch effects for downstream
  batch-effect assessment metrics like iLISI or kBET.
when_to_use_negative:
- When the goal is batch correction/removal rather than batch-effect detection and
  estimation — BEENE generates an embedding for assessing batch effects, not corrected
  expression data
- 'When a simple linear embedding suffices: the article positions BEENE for sophisticated
  non-linear batch effects that PC-space methods fail to capture'
- When biological predictor labels are required for the bio branch of the loss but
  none exist — the bio weight λ3 must be set to 0 with bio variables passed as None,
  otherwise training will fail
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: BEENE
  role: Provides the dezepe deep autoencoder class bee ne_model with get_hybrid_model_1,
    train_model, get_beene_embeddings, and evaluate_batch_iLisi for building, training,
    embedding, and evaluating
  repo: https://github.com/ashiq24/BEENE
- name: numpy
  role: Dense one-hot matrices, array handling, and saving embeddings via np.savetxt
- name: scikit-learn
  role: OneHotEncoder for batch labels and train_test_split for train/validation/test
    partitioning
- name: LISI
  role: Batch effect estimation metric (iLISI) computed on the BEENE embedding via
    evaluate_batch_iLisi
  repo: https://github.com/immunogenomics/LISI
- name: kBET
  role: R-based batch effect test; the learned embedding is transferred to R via rpy2
    for kBET calculation
  repo: https://github.com/theislab/kBET
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad479
    title: 'BEENE: deep learning-based nonlinear embedding improves batch effect estimation'
  source_content_hash: sha256:9a3b755bf9d76441092379ec852e94f77a89aa5571abc37a7ddf68d135808a2d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ashiq24/BEENE
  tools:
  - BEENE
  - LISI
  - kBET
  - numpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# single-cell-batch-effect-embedding-training

## Summary

Trains a BEENE deep nonlinear autoencoder that simultaneously learns batch and biological variables from RNA-seq data, producing a 50-dimensional embedding more robust and sensitive than PCA embedding for detecting and quantifying batch effects.

## When to use

Use when single-cell RNA-seq data from multiple batches requires a nonlinear lower-dimensional embedding that explicitly models batch effects for downstream batch-effect assessment metrics like iLISI or kBET.

## When NOT to use

- When the goal is batch correction/removal rather than batch-effect detection and estimation — BEENE generates an embedding for assessing batch effects, not corrected expression data
- When a simple linear embedding suffices: the article positions BEENE for sophisticated non-linear batch effects that PC-space methods fail to capture
- When biological predictor labels are required for the bio branch of the loss but none exist — the bio weight λ3 must be set to 0 with bio variables passed as None, otherwise training will fail

## Inputs

- 2D numpy gene expression matrix (rows = cells, columns = genes), e.g. mESC data from GSE65525
- Batch variable labels per cell (1D array for two categories, or one-hot dense matrix for >2 categories)
- Optional biological variable labels (one-hot dense matrix or 1D zeros/ones; None if absent)

## Outputs

- Trained BEENE autoencoder model (beene_model instance)
- BEENE embedding matrix, one row per cell, 50 columns, saved via np.savetxt to embedding.txt

## How to apply

Build the BEENE model with get_hybrid_model_1, specifying the number of genes, hidden layer dimensions (e.g. encoder 200→100 with SELU activations), a 50-dimensional embedding, the number of batch classes, and the three loss weights λ1 (reconstruction), λ2 (batch prediction), and λ3 (biological prediction). For high-dimensional data with thousands of genes and no prior dimensionality reduction, the README recommends λ1=0.001 and λ2=5.0 and setting islarge=True, which applies a higher dropout rate in the input layer; when features have been reduced to hundreds, use λ1=1, λ2=2 (and λ3=1 if biological labels exist). One-hot encode batch labels into a dense matrix with sklearn's OneHotEncoder, split data into train/validation/test sets with sklearn train_test_split, and call train_model (batch_size=32, up to 300 epochs), selecting the best-performing model by validation loss. Generate embeddings for held-out cells with get_beene_embeddings and save with np.savetxt for downstream iLISI/kBET evaluation.

## Related tools

- **BEENE** (Provides the dezepe deep autoencoder class bee ne_model with get_hybrid_model_1, train_model, get_beene_embeddings, and evaluate_batch_iLisi for building, training, embedding, and evaluating) — https://github.com/ashiq24/BEENE
- **numpy** (Dense one-hot matrices, array handling, and saving embeddings via np.savetxt)
- **scikit-learn** (OneHotEncoder for batch labels and train_test_split for train/validation/test partitioning)
- **LISI** (Batch effect estimation metric (iLISI) computed on the BEENE embedding via evaluate_batch_iLisi) — https://github.com/immunogenomics/LISI
- **kBET** (R-based batch effect test; the learned embedding is transferred to R via rpy2 for kBET calculation) — https://github.com/theislab/kBET

## Examples

```
my_model.get_hybrid_model_1(100,[50,20],5,3,2,1,2,1); my_model.train_model(X_train,Y_train,X_val,Y_val,300,32); emb = my_model.get_beene_embeddings(X_test); np.savetxt('embedding.txt', emb, fmt='%f')
```

## Evaluation signals

- embedding.txt exists, contains one row per held-out test cell with 50 columns of space-separated floats
- Validation loss shows converged reconstruction and batch-prediction components (best model selected by validation loss)
- When batch_weight λ2 is set higher than λ1 and λ3, embeddings should cluster more tightly by batch — i.e. higher iLISI batch mixing on the test set
- With bio variables absent, the biological predictor contributes zero loss (λ3=0, bio=None in train_model)
- iLISI computed on the random test split via evaluate_batch_iLisi returns one value per test sample

## Limitations

- Hyperparameter choice is data-dependent: README Table 1 recommends λ1=1, λ2=2 (features in hundreds, dimensionality reduction applied) versus λ1=0.001, λ2=5.0 (thousands of features, no reduction)
- kBET has no Python implementation and requires transferring the embedding to the R environment via rpy2 (R >= 4.1.0, rpy2 >= 3.4.0; Linux recommended)
- Batch variables for >2 categories must be provided as a one-hot dense matrix, not sparse
- islarge=True (higher input-layer dropout) is required for high-dimensional datasets to avoid overfitting

## Evidence

- [abstract] BEENE simultaneously learns the batch and biological variables from RNA-seq data, resulting in an embedding that is more robust and sensitive than PCA embedding: "BEENE simultaneously learns the batch and biological variables from RNA-seq data, resulting in an embedding that is more robust and sensitive than PCA embedding in terms of detecting and quantifying"
- [readme] get_hybrid_model_1 parameters: "get_hybrid_model_1(number_of_genes, hidden_layer_dimensions, embedding_dimension, number_of_batches, number_of_biological_variables, reconstruction_weight, batch_weight, bio_weight, islarge= False)"
- [readme] islarge dropout behavior: "islarge : bool, Set to true for the high dimensional dataset. The model uses a higher dropout rate in the input layer for high dimensional data."
- [readme] hyperparameter table for no dimensionality reduction: "if the dataset is used without employing any dimensionality reduction technique and contains thousands of features, it is recommended to decrease the value of \lambda1 accordingly (e.g., by a factor"
- [methods] train_model call in methods: "Training the model for 300 epochs
python
my_model.train_model(X_train,Y_Platform_train,Y_ER_train,X_val,Y_Platform_val,Y_ER_val,100)"
- [methods] embedding generation and storage: "test_embedding = my_model.get_beene_embeddings(X_test)
np.savetxt('embedding.txt', test_embedding, fmt='%f')"
- [intro] motivation for nonlinear embedding: "the batch effect estimation techniques mostly depend on the local distribution of cells in the Principal Component (PC) space, which may fail to capture sophisticated non-linear batch effects"
