---
name: beene-nonlinear-embedding-training
description: Use when you have a gene expression matrix (samples x genes) with batch
  labels (and optionally biological labels) and need a nonlinear embedding to detect
  or quantify batch effects, especially when PCA may miss nonlinear batch structure.
when_to_use_negative:
- When the analysis goal is batch correction rather than batch-effect detection/estimation
  — BEENE quantifies residual batch effects but does not itself correct them (correction
  tools like ComBat or Harmony are applied upstream).
- When only a linear embedding suffices and no batch/biological labels are available
  to supervise training — PCA is the simpler baseline in the paper.
- When kBET is the target metric and no R environment (R >= 4.1.0 with rpy2 >= 3.4.0,
  Linux recommended) is available, since kBET has no Python implementation.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: BEENE
  role: Core autoencoder (beene_model class) whose get_hybrid_model_1, train_model,
    get_beene_embeddings, and evaluate_batch_iLisi functions build, train, embed,
    and score the nonlinear embedding
  repo: https://github.com/ashiq24/BEENE
- name: scikit-learn
  role: OneHotEncoder for dense one-hot batch labels and train_test_split for train/validation/test
    partitioning
- name: numpy
  role: Array handling for expression matrices and np.savetxt storage of the learned
    embedding
- name: rpy2
  role: Transfers the learned BEENE embedding from Python to the R environment so
    kBET (R-only) can be computed
- name: kBET
  role: R package computing the k-nearest-neighbour batch-effect test rejection rate
    on the transferred BEENE embedding
  repo: https://github.com/theislab/kBET/
- name: LISI
  role: Local Inverse Simpson's Index metric underlying the evaluate_batch_iLisi batch-mixing
    score
  repo: https://github.com/immunogenomics/LISI
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad479
    title: 'BEENE: deep learning-based nonlinear embedding improves batch effect estimation'
  source_content_hash: sha256:2c8e4c372141bbc0e09be6e89f161910dd9c89b3ea7e2c01d1a0e2116994064d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ashiq24/BEENE
  tools:
  - BEENE
  - LISI
  - R
  - kBET
  - numpy
  - rpy2
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# beene-nonlinear-embedding-training

## Summary

Train the BEENE deep nonlinear autoencoder to produce a low-dimensional embedding of RNA-seq/single-cell data that captures batch and biological variables more sensitively than PCA space, for use in batch-effect estimation with iLISI and kBET.

## When to use

Use when you have a gene expression matrix (samples x genes) with batch labels (and optionally biological labels) and need a nonlinear embedding to detect or quantify batch effects, especially when PCA may miss nonlinear batch structure.

## When NOT to use

- When the analysis goal is batch correction rather than batch-effect detection/estimation — BEENE quantifies residual batch effects but does not itself correct them (correction tools like ComBat or Harmony are applied upstream).
- When only a linear embedding suffices and no batch/biological labels are available to supervise training — PCA is the simpler baseline in the paper.
- When kBET is the target metric and no R environment (R >= 4.1.0 with rpy2 >= 3.4.0, Linux recommended) is available, since kBET has no Python implementation.

## Inputs

- 2D numpy array gene expression matrix (rows = samples, columns = genes)
- 1D 0/1 batch label array (two categories) or dense one-hot matrix (>2 categories, via sklearn OneHotEncoder)
- Biological variable labels (1D 0/1 array or one-hot dense matrix; None if absent)
- Optional: batch-corrected expression matrices (e.g., ComBat- or Harmony-corrected)

## Outputs

- BEENE low-dimensional embedding (default embedding dimension 5) for the test set, e.g., saved as 'embedding.txt' via np.savetxt
- iLISI values per test-set sample from evaluate_batch_iLisi
- Trained beene_model object (get_hybrid_model_1) with tuned reconstruction/batch/bio weights
- Embedding transferred to R via rpy2 for kBET rejection-rate computation

## How to apply

Build the model with beene_model.get_hybrid_model_1(number_of_genes, hidden_layer_dimensions=[50,20], embedding_dimension=5, number_of_batches, number_of_biological_variables, reconstruction_weight, batch_weight, bio_weight), one-hot encoding the batch variable as a dense matrix when there are more than two categories (e.g., the three Chromium 10X PBMC protocols 3pV1, 3pV2, 5p) or using a 1D 0/1 array for two categories. Choose hyperparameters per Table 1: lambda_1=1, lambda_2=2, lambda_3=1 when dimensionality reduction reduced features to hundreds; if the data retain thousands of features without dimensionality reduction, decrease lambda_1 (e.g., by a factor of 1/1000 to 0.001) and set lambda_2=5, lambda_3=5. After a train/validation/test split, train with train_model for ~300 epochs (batch_size default 32), selecting the best model on the validation set. Generate embeddings for the test set with get_beene_embeddings and store them (np.savetxt to 'embedding.txt'); a higher batch_weight makes the embedding more sensitive to batch effects. Because kBET is R-only, transfer the learned embedding to R via rpy2 (rpy2 >= 3.4.0, R >= 4.1.0); iLISI is computed in Python via evaluate_batch_iLisi(data, batch_var, bio_var, seed) with a fixed seed.

## Related tools

- **BEENE** (Core autoencoder (beene_model class) whose get_hybrid_model_1, train_model, get_beene_embeddings, and evaluate_batch_iLisi functions build, train, embed, and score the nonlinear embedding) — https://github.com/ashiq24/BEENE
- **scikit-learn** (OneHotEncoder for dense one-hot batch labels and train_test_split for train/validation/test partitioning)
- **numpy** (Array handling for expression matrices and np.savetxt storage of the learned embedding)
- **rpy2** (Transfers the learned BEENE embedding from Python to the R environment so kBET (R-only) can be computed)
- **kBET** (R package computing the k-nearest-neighbour batch-effect test rejection rate on the transferred BEENE embedding) — https://github.com/theislab/kBET/
- **LISI** (Local Inverse Simpson's Index metric underlying the evaluate_batch_iLisi batch-mixing score) — https://github.com/immunogenomics/LISI

## Examples

```
my_model.get_hybrid_model_1(100,[50,20],5,3,2,1,2,1); my_model.train_model(X_train,Y_batch_train,Y_bio_train,X_val,Y_batch_val,Y_bio_val,300); test_embedding = my_model.get_beene_embeddings(X_test); np.savetxt('embedding.txt', test_embedding, fmt='%f')
```

## Evaluation signals

- On ComBat-corrected mESC data, BEENE kBET rejection rate should be higher (more sensitive) than PCA, e.g., 0.58 vs 0.33/0.20, with BEENE iLISI ~1.57 vs ~1.76 in PCA space
- On Harmony-corrected PBMC data (3pV1/3pV2/5p batches), BEENE kBET rejection rate should drop to ~0.80 versus ~1.00 for uncorrected and ComBat-corrected data (PCA ComBat ~0.96)
- Median iLISI should rise from ~1 to ~2.5 on BEENE embeddings after Harmony correction, matching the Fig. 4 pattern
- The trained model should show lower reconstruction/batch/bio prediction error on the validation set than on training epochs, since best model is selected by validation performance
- Output embedding file should have one row per test sample and embedding_dimension columns (e.g., 5), in the format written by np.savetxt with fmt='%f'

## Limitations

- Hyperparameter choice is data-dependent: with batch_weight set comparatively higher, embeddings cluster more tightly by batch, making BEENE more sensitive to batch effects; wrong weights distort the embedding
- For high-dimensional data without prior dimensionality reduction, lambda_1 must be decreased (e.g., 0.001) or training is dominated by reconstruction loss
- kBET requires transferring embeddings to R (R >= 4.1.0, rpy2 >= 3.4.0); Linux is recommended for seamless rpy2 operation
- kBET rejection rate of 1 on batch-corrected data may warrant supplementary measures such as average silhouette width or PCA-based batch-effect measures
- batch and bio variables with more than two categories must be one-hot encoded as dense matrices; 1D 0/1 arrays are only valid for two categories

## Evidence

- [methods] model construction: "get_hybrid_model_1(number_of_genes, hidden_layer_dimensions, embedding_dimension,"
- [methods] training: "Training the model for 300 epochs
python
my_model.train_model(X_train,Y_Platform_train,Y_ER_train,X_val,Y_Platform_val,Y_ER_val,100)"
- [methods] embedding storage: "test_embedding = my_model.get_beene_embeddings(X_test)
np.savetxt('embedding.txt', test_embedding, fmt='%f')"
- [methods] iLISI computation: "lisi_values = my_model.evaluate_batch_iLisi(Xt,bt,yt,20)"
- [methods] kBET transfer: "To calculate kBET using non-linear embedding, the learned embedding must be transferred to R environment for calculation."
- [readme] hyperparameter guidance: "We recommend to use  $\lambda_1 = \lambda_3 = 1$ and $\lambda_2 = 2$ when a dimensionality reduction technique has been used to reduce the number of features to hundreds."
- [readme] sensitivity rationale: "BEENE will be more sensetive to the batch effect if the *batch_weight* parameter is set higher."
- [readme] data splitting: "The data is split into test, training, and validation
      set. The model defined by the user is trained by the training set. The best-performing model
      is selected by the validation set."
