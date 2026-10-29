---
name: single-cell-batch-effect-quantification
description: Use when single-cell RNA-seq data span multiple batches (e.g. 10X 3pV1/3pV2/5p,
  five islet platforms inDrop/Fluidigm C1/Smart-Seq2/CEL-seq/CEL-seq2) and you need
  rejection-rate and mixing metrics in both PCA and nonlinear embedding space.
when_to_use_negative:
- Input is not gene-expression count/normalized matrix with batch labels — BEENE requires
  gene expression per sample plus batch and (optionally) biological variables
- You need batch effect correction (e.g. producing corrected data); this skill only
  estimates/quantifies the extent of batch effects — correction is done by methods
  like ComBat or Harmony
- You need a fast linear-only assessment on already-reduced low-dimensional features
  where a PCA-space metric is known to suffice
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: BEENE
  role: Deep nonlinear autoencoder (beene_model class) that builds, trains, embeds
    (get_hybrid_model_1, train_model, get_beene_embeddings) and computes iLISI (evaluate_batch_iLisi)
  repo: https://github.com/ashiq24/BEENE
- name: kBET
  role: R-only metric computing average Pearson chi-square test rejection rate of
    batch mixing on transferred embeddings
  repo: https://github.com/theislab/kBET
- name: LISI
  role: Local Inverse Simpson's Index metric underlying the iLISI batch-mixing values
    computed on the BEENE embedding
  repo: https://github.com/immunogenomics/LISI
- name: rpy2
  role: Bridges Python to R (>= 4.1.0) so the learned BEENE embedding can be transferred
    to the R environment for kBET calculation
- name: scikit-learn
  role: OneHotEncoder for dense one-hot batch labels, train_test_split, and PCA-space
    baseline embedding
- name: numpy
  role: Array handling and np.savetxt export of embeddings to embedding.txt
- name: Python
  role: Primary execution environment (>= 3.7.0) hosting the BEENE model and metric
    pre-processing
- name: R
  role: Environment (>= 4.1.0) in which kBET runs on transferred embeddings
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad479
    title: 'BEENE: deep learning-based nonlinear embedding improves batch effect estimation'
  source_content_hash: sha256:b0cbf72e1cdaced9aa6764f9d26f1e60da9c2045e1673adf4ae9993438a17ced
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ashiq24/BEENE
  tools:
  - BEENE
  - LISI
  - Python
  - R
  - kBET
  - numpy
  - rpy2
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# single-cell-batch-effect-quantification

## Summary

Quantify batch effects in scRNA-seq/RNA-seq data by training a BEENE nonlinear autoencoder embedding and comparing it against a PCA baseline using kBET rejection rates (via rpy2/R) and iLISI values, so non-linear batch structure is detected where PC-space metrics fail.

## When to use

Use when single-cell RNA-seq data span multiple batches (e.g. 10X 3pV1/3pV2/5p, five islet platforms inDrop/Fluidigm C1/Smart-Seq2/CEL-seq/CEL-seq2) and you need rejection-rate and mixing metrics in both PCA and nonlinear embedding space.

## When NOT to use

- Input is not gene-expression count/normalized matrix with batch labels — BEENE requires gene expression per sample plus batch and (optionally) biological variables
- You need batch effect correction (e.g. producing corrected data); this skill only estimates/quantifies the extent of batch effects — correction is done by methods like ComBat or Harmony
- You need a fast linear-only assessment on already-reduced low-dimensional features where a PCA-space metric is known to suffice

## Inputs

- Processed scRNA-seq gene expression matrix (2D numpy array, rows = cells/samples, columns = genes), e.g. from harmony2019 figure4/figure5 deposits
- Batch variable labels (one-hot dense matrix if >2 categories, 1D 0/1 array if 2 categories)
- Biological variable labels (optional, same encoding rules)
- Hyperparameter choices from Table 1 (lambda1, lambda2, lambda3, islarge)

## Outputs

- BEENE embedding matrix (embedding_dim=5 in the examples) written as space-separated embedding.txt via np.savetxt
- Per-sample iLISI values on the held-out test set
- Per-batch kBET average rejection rates for PCA and BEENE embeddings
- Comparison table (e.g. batch_metrics_table.csv / batch_metrics.csv) of kBET rejection rates and median iLISI for PCA vs BEENE

## How to apply

One-hot encode multi-category batch labels as dense matrices with sklearn's OneHotEncoder (two-category variables stay as 1D 0/1 arrays), build the BEENE model with get_hybrid_model_1 using Table 1 hyperparameters — e.g. lambda1=0.001, lambda2=5.0, lambda3=5.0 for thousands of features without dimensionality reduction, or lambda1=1, lambda2=2, lambda3=1 after reducing to hundreds of features — and set islarge=True for high-dimensional input dropout. Split data into training-validation-test sets, train with train_model (e.g. 300 epochs), and export embeddings with get_beene_embeddings plus np.savetxt('embedding.txt'). Compute iLISI per sample with evaluate_batch_iLisi(data, batch_var, bio_var, seed) on the held-out test set, and transfer the embedding to R via rpy2 to run kBET, which returns an average test rejection rate (lower = less batch bias). Assemble kBET rejection rates and median iLISI per batch for PCA vs BEENE spaces into a comparison table and validate against reported patterns (e.g. kBET ~1.00 uncorrected/ComBat-corrected, BEENE dropping to ~0.80 after Harmony, iLISI rising from ~1 to ~2.5).

## Related tools

- **BEENE** (Deep nonlinear autoencoder (beene_model class) that builds, trains, embeds (get_hybrid_model_1, train_model, get_beene_embeddings) and computes iLISI (evaluate_batch_iLisi)) — https://github.com/ashiq24/BEENE
- **kBET** (R-only metric computing average Pearson chi-square test rejection rate of batch mixing on transferred embeddings) — https://github.com/theislab/kBET
- **LISI** (Local Inverse Simpson's Index metric underlying the iLISI batch-mixing values computed on the BEENE embedding) — https://github.com/immunogenomics/LISI
- **rpy2** (Bridges Python to R (>= 4.1.0) so the learned BEENE embedding can be transferred to the R environment for kBET calculation)
- **scikit-learn** (OneHotEncoder for dense one-hot batch labels, train_test_split, and PCA-space baseline embedding)
- **numpy** (Array handling and np.savetxt export of embeddings to embedding.txt)
- **Python** (Primary execution environment (>= 3.7.0) hosting the BEENE model and metric pre-processing)
- **R** (Environment (>= 4.1.0) in which kBET runs on transferred embeddings)

## Examples

```
my_model.get_hybrid_model_1(100,[50,20],5,3,2,1,2,1); my_model.train_model(X_train,Y_Platform_train,Y_ER_train,X_val,Y_Platform_val,Y_ER_val,100)
```

## Evaluation signals

- All kBET rejection-rate values fall in [0, 1]; median iLISI values fall in [1, 5]
- batch_metrics_table.csv exists and contains kBET rejection-rate and iLISI entries for both PCA and BEENE embeddings for each batch
- PBMC reference pattern reproduced: kBET ~1.00 for uncorrected and ComBat-corrected data (PCA ComBat ~0.96), BEENE drops to ~0.80 after Harmony; median iLISI rises from ~1 to ~2.5 on BEENE embeddings after Harmony
- Five-batch islet analysis reproduces Supplementary Section 2.6 pattern across inDrop, Fluidigm C1, Smart-Seq2, CEL-seq, CEL-seq2 batches
- A hyperparameter row from Table 1 matching the feature count regime (hundreds vs thousands of genes) was used, with islarge=True for high-dimensional input

## Limitations

- kBET has no Python implementation and is available only in R, requiring rpy2 (>= 3.4.0) and R (>= 4.1.0); Linux is recommended for rpy2
- If kBET returns an average rejection rate of 1 for batch-corrected data, complementary measures (average silhouette width, PCA-based batch-effect measures) should be considered
- Hyperparameter choice is data-dependent: lambda2 higher than lambda1/lambda3 makes BEENE more sensitive to batch effects; for thousands of features without dimensionality reduction lambda1 should be decreased (e.g. by a factor of 1/1000)
- Existing PC-space-based batch effect estimation may fail to capture sophisticated non-linear batch effects — which is the motivation for BEENE, but PCA baselines are still computed for comparison

## Evidence

- [readme] BEENE hyperparameter guideline: "We recommend to use $\lambda_1 = \lambda_3 = 1$ and $\lambda_2 = 2$ when a dimensionality reduction technique has been used to reduce the number of features to hundreds."
- [readme] kBET output definition: [evidence span withheld — anchor status: external_doc]
- [abstract] BEENE trains batch/bio-aware embedding: "BEENE simultaneously learns the batch and biological variables from RNA-seq data, resulting in an embedding that is more robust and sensitive than PCA embedding in terms of detecting and quantifying"
- [methods] kBET transfer workflow: "To calculate kBET using non-linear embedding, the learned embedding must be transferred to R environment for calculation."
- [methods] iLISI evaluation function: "evaluate_batch_iLisi(data, batch_var, bio_var, seed):"
- [methods] Embedding export: "np.savetxt('embedding.txt', test_embedding, fmt='%f')"
- [methods] Training procedure: "Training the model for 300 epochs"
- [readme] One-hot encoding rule: "For more than two categories, it must be one hot representation of batch labels for each of the samples in the data and must be a dense matrix."
