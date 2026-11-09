---
name: rna-seq-dimensionality-reduction
description: Use when you have a gene-by-cell RNA-seq expression matrix (thousands
  of genes) with batch labels and need a lower-dimensional embedding that preserves
  batch-specific variation for batch effect detection/quantification.
when_to_use_negative:
- Input features have already been reduced to a few hundreds by another dimensionality
  reduction technique — use the λ1=λ3=1, λ2=2 hyperparameter regime rather than the
  high-dimensional settings, and PCA may suffice
- The goal is batch effect removal/correction of the expression data rather than detection/quantification
  of batch effects via an embedding
- Only linear batch effects need assessment on local cell distributions in PC space
  — simpler PCA-based batch estimation may be adequate
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: BEENE
  role: Provides the _beene_model_ class (get_hybrid_model_1, train_model, get_beene_embeddings)
    that builds, trains, and extracts the nonlinear auto-encoder embedding
  repo: https://github.com/ashiq24/BEENE
- name: scikit-learn
  role: OneHotEncoder for dense one-hot batch encoding and train_test_split for train/validation/test
    splits
- name: numpy
  role: Array handling for expression matrices and np.savetxt storage of the embedding
- name: LISI
  role: Optional downstream metric (iLISI via evaluate_batch_iLisi) to assess batch
    mixing on the learned embedding
  repo: https://github.com/immunogenomics/LISI
- name: kBET
  role: Optional R-only batch effect metric; the learned embedding is transferred
    to R via rpy2 for kBET calculation
  repo: https://github.com/theislab/kBET/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad479
    title: 'BEENE: deep learning-based nonlinear embedding improves batch effect estimation'
  source_content_hash: sha256:bc8a14ef5686f0504b2220959409733aaa3567d28c3721f3d7f60019004f6e54
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# rna-seq-dimensionality-reduction

## Summary

Generate a low-dimensional nonlinear BEENE embedding from RNA-seq data using a deep auto-encoder that simultaneously learns batch and biological variables, suitable for detecting both linear and nonlinear batch effects where PCA-based embeddings fail.

## When to use

Use when you have a gene-by-cell RNA-seq expression matrix (thousands of genes) with batch labels and need a lower-dimensional embedding that preserves batch-specific variation for batch effect detection/quantification.

## When NOT to use

- Input features have already been reduced to a few hundreds by another dimensionality reduction technique — use the λ1=λ3=1, λ2=2 hyperparameter regime rather than the high-dimensional settings, and PCA may suffice
- The goal is batch effect removal/correction of the expression data rather than detection/quantification of batch effects via an embedding
- Only linear batch effects need assessment on local cell distributions in PC space — simpler PCA-based batch estimation may be adequate

## Inputs

- 2D numpy array of gene expression values (one row per cell/sample, one column per gene)
- Batch variable labels per sample (1D array of 0/1 for two categories; dense one-hot matrix for >2 categories, from sklearn OneHotEncoder)
- Optional biological variable labels (1D array of zeros/ones for two categories, or dense one-hot matrix; None if absent)
- Number of genes per sample (number_of_genes) and number of batch/biological classes

## Outputs

- BEENE embedding matrix (one row per cell, embedding_dimension columns, e.g. 50 dims) written as embedding.txt via np.savetxt with fmt='%f'
- Trained _beene_model_ object with converged reconstruction and batch-prediction losses

## How to apply

Load the gene expression matrix (rows = cells, columns = genes) with batch labels and split it with sklearn train_test_split (test_size=0.20, then test_size=0.25 of the remainder as validation). One-hot encode the batch variables with sklearn's OneHotEncoder into a dense matrix. Build the model with get_hybrid_model_1: for high-dimensional data (thousands of genes, no prior dimensionality reduction, no biological variable) use reconstruction_weight λ1=0.001 and batch_weight λ2=5.0, set islarge=True so the input layer uses a higher dropout rate, and choose the encoder/decoder hidden layer dimensions and embedding dimension (e.g., 200→100 encoder, 100→200 decoder, 50-dimensional latent embedding). Train with train_model (batch_size=32, ~300 epochs) while monitoring validation loss to select the best model, then extract embeddings for held-out test cells with get_beene_embeddings and save them via np.savetxt to embedding.txt (space-separated floats, one row per cell). If a prior dimensionality reduction reduced features to hundreds, use the README guideline λ1=λ3=1, λ2=2 instead.

## Related tools

- **BEENE** (Provides the _beene_model_ class (get_hybrid_model_1, train_model, get_beene_embeddings) that builds, trains, and extracts the nonlinear auto-encoder embedding) — https://github.com/ashiq24/BEENE
- **scikit-learn** (OneHotEncoder for dense one-hot batch encoding and train_test_split for train/validation/test splits)
- **numpy** (Array handling for expression matrices and np.savetxt storage of the embedding)
- **LISI** (Optional downstream metric (iLISI via evaluate_batch_iLisi) to assess batch mixing on the learned embedding) — https://github.com/immunogenomics/LISI
- **kBET** (Optional R-only batch effect metric; the learned embedding is transferred to R via rpy2 for kBET calculation) — https://github.com/theislab/kBET/

## Examples

```
test_embedding = my_model.get_beene_embeddings(X_test); np.savetxt('embedding.txt', test_embedding, fmt='%f')
```

## Evaluation signals

- embedding.txt exists and contains one row per test cell with exactly the chosen embedding dimension (e.g. 50) space-separated float columns
- Validation loss (joint reconstruction + batch-prediction) converges over ~300 epochs and the best model is selected on the validation split
- Embeddings from get_beene_embeddings match the number of rows of the held-out test expression matrix
- iLISI values computed via evaluate_batch_iLisi on the BEENE embedding are more sensitive/robust than iLISI computed on PCA embedding
- Hyperparameters match the README Table 1 guideline for the data regime (λ1=0.001, λ2=5.0, no λ3 for thousands of features without biological variables)

## Limitations

- When batch_weight is set higher than reconstruction_weight and bio_weight, embeddings cluster more tightly by batch — hyperparameter choice is data- and downstream-application-dependent (README Table 1)
- For datasets with thousands of features and no dimensionality reduction, λ1 must be decreased accordingly (e.g., by a factor of 1/1000), otherwise reconstruction loss dominates
- islarge=True (higher input-layer dropout) is required for high-dimensional datasets; omitting it risks poor training on high-dimensional data
- Existing batch effect estimation techniques depending on local distributions in PC space may fail to capture sophisticated non-linear batch effects — PCA embeddings are an inadequate baseline in that regime

## Evidence

- [intro] BEENE is the first known method to explicitly consider and model the non-linearity of batch effects for effective batch effect detection and estimation.: "BEENE is the first known method to explicitly consider and model the non-linearity of batch effects for effective batch effect detection and estimation."
- [readme] get_hybrid_model_1 signature and islarge parameter: "islarge : bool, Set to true for the high dimensional dataset. The model uses a higher dropout rate in the input layer for high dimensional data."
- [readme] Hyperparameter guideline: "We recommend to use $\lambda_1 = \lambda_3 = 1$ and $\lambda_2 = 2$ when a dimensionality reduction technique has been used to reduce the number of features to hundreds."
- [methods] Embedding generation and storage: "test_embedding = my_model.get_beene_embeddings(X_test)
np.savetxt('embedding.txt', test_embedding, fmt='%f')"
- [methods] Training call: "my_model.train_model(X_train,Y_Platform_train,Y_ER_train,X_val,Y_Platform_val,Y_ER_val,100)"
