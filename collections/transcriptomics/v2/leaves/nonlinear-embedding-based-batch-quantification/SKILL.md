---
name: nonlinear-embedding-based-batch-quantification
description: Use when detecting or quantifying the extent of batch effects introduced
  by technical non-biological factors in RNA-seq data, especially when existing PCA-based
  estimation may fail to capture non-linear batch effects.
when_to_use_negative:
- When the goal is to REMOVE/correct batch effects (e.g., with ComBat, ComBat-Seq,
  Harmony, or MNN) rather than estimate/detect their extent — BEENE is an estimation,
  not correction, method
- When batch labels or covariate annotations for each sample are unavailable, since
  batch prediction losses and iLISI require batch/biological variables
- When only a small feature count after dimensionality reduction is available and
  the lambda1=0.001 / lambda2=5 / lambda3=5 high-dimensional hyperparameter regime
  would be inapplicable without adjusting weights per Table 1
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: BEENE
  role: Deep nonlinear auto-encoder that generates the lower-dimensional embedding
    used for batch effect estimation; provides get_hybrid_model_1, train_model, get_beene_embeddings,
    and evaluate_batch_iLisi
  repo: https://github.com/ashiq24/BEENE
- name: LISI
  role: Batch effect estimation metric providing iLISI computed on the BEENE embedding
  repo: https://github.com/immunogenomics/LISI
- name: kBET
  role: R-only k-nearest-neighbour batch effect test computing the average rejection
    rate on the BEENE embedding transferred from Python via rpy2
  repo: https://github.com/theislab/kBET
- name: rpy2
  role: Transfers the learned BEENE embedding from the Python environment to R for
    kBET calculation (rpy2 >= 3.4.0)
- name: numpy
  role: Provides the 2D gene-expression arrays and one-hot encoding inputs required
    by beene_model functions
- name: scikit-learn
  role: Used with numpy for OneHotEncoder of batch/biological variables and train_test_split
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad479
    title: 'BEENE: deep learning-based nonlinear embedding improves batch effect estimation'
  source_content_hash: sha256:1b3250f1da874f9bc1676473944d39a8f62563326803e290a804341a748c8675
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# nonlinear-embedding-based-batch-quantification

## Summary

Quantify batch effects in scRNA-seq/RNA-seq data by computing iLISI and kBET metrics on BEENE's deep nonlinear auto-encoder embedding rather than on PCA space, which can miss sophisticated non-linear batch effects.

## When to use

Use when detecting or quantifying the extent of batch effects introduced by technical non-biological factors in RNA-seq data, especially when existing PCA-based estimation may fail to capture non-linear batch effects.

## When NOT to use

- When the goal is to REMOVE/correct batch effects (e.g., with ComBat, ComBat-Seq, Harmony, or MNN) rather than estimate/detect their extent — BEENE is an estimation, not correction, method
- When batch labels or covariate annotations for each sample are unavailable, since batch prediction losses and iLISI require batch/biological variables
- When only a small feature count after dimensionality reduction is available and the lambda1=0.001 / lambda2=5 / lambda3=5 high-dimensional hyperparameter regime would be inapplicable without adjusting weights per Table 1

## Inputs

- 2D numpy array of gene expression data (rows = samples, columns = genes)
- Batch variable per sample (1D array of 0/1 for two categories, dense one-hot matrix for more)
- Biological variable per sample (same encoding as batch; None if absent)
- Random seed int for train/validation/test split

## Outputs

- BEENE low-dimensional embedding per sample (savable with np.savetxt to embedding.txt)
- iLISI values for each sample in the randomly chosen test set
- kBET average rejection rate computed in R on the transferred embedding

## How to apply

Build a BEENE model with beene_model.get_hybrid_model_1(number_of_genes, hidden_layer_dimensions, embedding_dimension, number_of_batches, number_of_biological_variables, reconstruction_weight, batch_weight, bio_weight), one-hot encode the batch variables, and create a training-validation-test split with a fixed seed. Train the model for 300 epochs (train_model with batch_size default 32), then obtain the embedding via get_beene_embeddings and compute iLISI with evaluate_batch_iLisi(data, batch_var, bio_var, seed), which returns iLISI per sample in the random test set. For kBET (R-only, no Python implementation), transfer the learned embedding to R via rpy2 >= 3.4.0 and run the R kBET package, which returns the average test rejection rate — the lower the rejection rate, the less batch bias. Choose hyperparameters per the README's Table 1: with biological features present and dimensionality reduction applied (features in the hundreds), use lambda1=1, lambda2=2, lambda3=1; without feature reduction (thousands of features), use lambda1=0.001, lambda2=5, lambda3=5. Higher batch_weight relative to the other weights makes BEENE more sensitive to batch effects.

## Related tools

- **BEENE** (Deep nonlinear auto-encoder that generates the lower-dimensional embedding used for batch effect estimation; provides get_hybrid_model_1, train_model, get_beene_embeddings, and evaluate_batch_iLisi) — https://github.com/ashiq24/BEENE
- **LISI** (Batch effect estimation metric providing iLISI computed on the BEENE embedding) — https://github.com/immunogenomics/LISI
- **kBET** (R-only k-nearest-neighbour batch effect test computing the average rejection rate on the BEENE embedding transferred from Python via rpy2) — https://github.com/theislab/kBET
- **rpy2** (Transfers the learned BEENE embedding from the Python environment to R for kBET calculation (rpy2 >= 3.4.0))
- **numpy** (Provides the 2D gene-expression arrays and one-hot encoding inputs required by beene_model functions)
- **scikit-learn** (Used with numpy for OneHotEncoder of batch/biological variables and train_test_split)

## Examples

```
my_model.get_hybrid_model_1(100,[50,20],5,3,2,1,2,1); my_model.train_model(X_train,Y_Platform_train,Y_ER_train,X_val,Y_Platform_val,Y_ER_val,100); lisi_values = my_model.evaluate_batch_iLisi(Xt,bt,yt,20)
```

## Evaluation signals

- The trained BEENE model is selected by the validation split and iLISI is only reported on the held-out test split with the fixed seed — verify the split was performed before trusting iLISI values
- Batch variable encoding matches the function contract: 1D zeros/ones for two categories, dense one-hot matrix for >2 categories, else evaluate_batch_iLisi fails
- Lower kBET average rejection rate indicates less batch bias; a rejection rate of 1 on corrected data signals the metric is saturated and complementary measures (silhouette width, PCA-based) should be checked
- Hyperparameters match README Table 1 for the feature regime used (e.g., lambda1=0.001, lambda2=5, lambda3=5 when thousands of features and no dimensionality reduction)
- Embedding file round-trips correctly: np.savetxt('embedding.txt', test_embedding, fmt='%f') and the R-side kBET run consumes the same matrix

## Limitations

- kBET has no Python implementation and is available only in R, requiring rpy2 (>= 3.4.0) transfer; Linux system is recommended for seamless rpy2 operation
- Hyperparameter choice (reconstruction_weight, batch_weight, bio_weight) is data- and application-dependent; wrong weights change sensitivity — e.g., a higher batch_weight clusters cells more tightly by batch
- For high-dimensional datasets (thousands of features, no dimensionality reduction), reconstruction weight must be decreased (e.g., by a factor of 1/1000) or reconstruction dominates; islarge=True enables higher input-layer dropout
- If kBET returns an average rejection rate of 1 on batch-corrected data, it is too sensitive and additional measures (average silhouette width, PCA-based batch-effect measures) should be explored
- Two-category batch/biological variables must be 1D arrays, not one-hot, or the functions will misbehave

## Evidence

- [intro] BEENE is the first known method to explicitly consider and model the non-linearity of batch effects for effective batch effect detection and estimation.: "BEENE is the first known method to explicitly consider and model the non-linearity of batch effects for effective batch effect detection and estimation."
- [methods] lisi computation via evaluate_batch_iLisi: "lisi_values = my_model.evaluate_batch_iLisi(Xt,bt,yt,20)"
- [methods] training 300 epochs: "Training the model for 300 epochs"
- [methods] rpy2 requirement for kBET: "rpy2 >= 3.4.0 (For seamless working this package Linux system is recommended)"
- [readme] README hyperparameter guideline: "We recommend to use  $\lambda_1 = \lambda_3 = 1$ and $\lambda_2 = 2$ when a dimensionality reduction technique has been used to reduce the number of features to hundreds."
- [readme] kBET interpretation: [evidence span withheld — anchor status: external_doc]
- [methods] embedding storage: "test_embedding = my_model.get_beene_embeddings(X_test)
np.savetxt('embedding.txt', test_embedding, fmt='%f')"
