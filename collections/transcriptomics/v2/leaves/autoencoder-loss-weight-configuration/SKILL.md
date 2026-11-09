---
name: autoencoder-loss-weight-configuration
description: Use when building or retuning a BEENE get_hybrid_model_1 autoencoder
  for RNA-seq batch-effect estimation and you must choose λ1/λ2/λ3 and dropout based
  on feature count, dimensionality reduction, and biological variables.
when_to_use_negative:
- The downstream goal is batch correction/removal rather than detection and estimation
  of batch effects — BEENE is an estimation technique whose embedding clusters by
  batch when batch_weight is high
- Features are already reduced to a few hundred via a dimensionality-reduction technique
  but you set λ1=0.001, which the Table 1 guideline reserves for thousands of unreduced
  features
- The batch variable has no label information — λ2 requires supervised batch-label
  prediction to tune the embedding
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BEENE
  role: Provides the get_hybrid_model_1 constructor whose reconstruction_weight, batch_weight,
    and bio_weight parameters are configured by this skill
  repo: https://github.com/ashiq24/BEENE
- name: scikit-learn
  role: One-hot encodes batch variables (OneHotEncoder) and creates train/validation/test
    splits before model training
- name: numpy
  role: Holds the gene-expression matrix and dense one-hot batch vectors; np.savetxt
    stores the resulting embedding.txt
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad479
    title: 'BEENE: deep learning-based nonlinear embedding improves batch effect estimation'
  source_content_hash: sha256:d0f3bdcc50077c8c052517449d15131781ad09b4334b637937be85dda51b37b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ashiq24/BEENE
  tools:
  - BEENE
  - numpy
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# autoencoder-loss-weight-configuration

## Summary

Guidance for setting the joint-loss weights (reconstruction_weight λ1, batch_weight λ2, bio_weight λ3) of the BEENE deep autoencoder, plus input-layer dropout, so the learned embedding balances reconstruction quality against batch-effect sensitivity.

## When to use

Use when building or retuning a BEENE get_hybrid_model_1 autoencoder for RNA-seq batch-effect estimation and you must choose λ1/λ2/λ3 and dropout based on feature count, dimensionality reduction, and biological variables.

## When NOT to use

- The downstream goal is batch correction/removal rather than detection and estimation of batch effects — BEENE is an estimation technique whose embedding clusters by batch when batch_weight is high
- Features are already reduced to a few hundred via a dimensionality-reduction technique but you set λ1=0.001, which the Table 1 guideline reserves for thousands of unreduced features
- The batch variable has no label information — λ2 requires supervised batch-label prediction to tune the embedding

## Inputs

- 2D numpy array of gene expression (rows = cells/samples, columns = genes)
- Batch labels as a numpy array (one-hot dense matrix for >2 categories, 1D 0/1 array for two)
- Biological variable labels (one-hot dense matrix or 1D array; None if absent)
- Number of genes, number of batch classes, number of biological classes

## Outputs

- Configured and trained BEENE model (beene_model class instance)
- 50-dimensional BEENE embedding per test cell, savable via np.savetxt to embedding.txt

## How to apply

Instantiate the model with get_hybrid_model_1(number_of_genes, hidden_layer_dimensions, embedding_dimension, number_of_batches, number_of_biological_variables, reconstruction_weight, batch_weight, bio_weight, islarge) and set the three loss weights from the BEENE Table 1 guidelines: λ1=λ3=1, λ2=2 when features were reduced to a few hundreds and the biological feature is present; λ1=0.001, λ2=5.0, λ3=5.0 when thousands of raw features are used with a biological feature present; drop λ3 when no biological variables exist (λ1=1.0/λ2=1.0 with DR, λ1=0.001/λ2=5.0 without). If the dataset is used without dimensionality reduction and contains thousands of features, decrease λ1 by a factor of 1/1000. Rationale: a relatively higher batch_weight (λ2) makes cell embeddings cluster more tightly by batch, increasing BEENE's sensitivity to batch effects, while λ1 controls reconstruction of the autoencoder. For high-dimensional input set islarge=True so the input layer applies a higher dropout rate (e.g., 0.7 for the mESC data); omit the biological predictor entirely by passing bio variables as None with λ3=0. Train with train_model (batch_size=32, ~300 epochs) and select the best model by validation loss.

## Related tools

- **BEENE** (Provides the get_hybrid_model_1 constructor whose reconstruction_weight, batch_weight, and bio_weight parameters are configured by this skill) — https://github.com/ashiq24/BEENE
- **scikit-learn** (One-hot encodes batch variables (OneHotEncoder) and creates train/validation/test splits before model training)
- **numpy** (Holds the gene-expression matrix and dense one-hot batch vectors; np.savetxt stores the resulting embedding.txt)

## Examples

```
my_model.get_hybrid_model_1(number_of_genes, [200,100], 50, number_of_batches, number_of_bio, 0.001, 5.0, 0, islarge=True)
```

## Evaluation signals

- embedding.txt exists with one row per held-out test cell and exactly 50 columns (the embedding_dimension set in get_hybrid_model_1)
- Validation reconstruction and batch-prediction losses converge over ~300 epochs of train_model (batch_size=32), and the best model is selected by validation loss
- Weights match Table 1 conditions: e.g., λ1=0.001/λ2=5.0 with no dimensionality reduction (thousands of genes), λ1=1/λ2=2/λ3=1 after reduction to hundreds with a biological feature present
- Raising batch_weight relative to λ1/λ3 visibly tightens embedding clustering by batch labels (iLISI on test set via evaluate_batch_iLisi)

## Limitations

- λ1, λ2, and λ3 choice is data- and downstream-application-dependent; Table 1 is a guideline, not a universal rule
- bio_weight is inapplicable when biological variables are absent (λ3 must be dropped, bio variables passed as None)
- For high-dimensional data without dimensionality reduction, λ1 must be scaled down (e.g., 1/1000) or reconstruction will dominate; islarge=True should be used to raise input-layer dropout
- kBET requires transferring the learned embedding to R via rpy2 since no Python implementation exists

## Evidence

- [readme] Table 1 guideline for λ with dimensionality reduction: "We recommend to use $\lambda_1 = \lambda_3 = 1$ and $\lambda_2 = 2$ when a dimensionality reduction technique has been used to reduce the number of features to hundreds."
- [readme] Scaling λ1 without dimensionality reduction: "if the dataset is used without employing any dimensionality reduction technique and contains thousands of features, it is recommended to decrease the value of \lambda1 accordingly (e.g., by a factor"
- [readme] Effect of batch_weight on embedding sensitivity: "BEENE will be more sensetive to the batch effect if the *batch_weight* parameter is set higher."
- [methods] get_hybrid_model_1 parameter roles: "reconstruction_weight: float, weight for auto-encoder reconstruction loss"
- [methods] Concrete instantiation example: "my_model.get_hybrid_model_1(100,[50,20],5,3,2,1,2,1)"
- [methods] Training with validation monitoring: "my_model.train_model(X_train,Y_Platform_train,Y_ER_train,X_val,Y_Platform_val,Y_ER_val,100)"
