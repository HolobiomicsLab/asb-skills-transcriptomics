---
name: pca-vs-nonlinear-embedding-batch-benchmarking
description: Use when quantifying batch effects in single-cell RNA-seq data where
  PCA-space metrics may miss nonlinear batch structure — e.g., comparing uncorrected
  vs. ComBat/Harmony-corrected data with kBET and iLISI.
when_to_use_negative:
- When the goal is batch correction itself rather than quantifying/estimating batch
  effect extent — BEENE estimates effects, it does not remove them
- When the embedding input is a high-dimensional dataset with thousands of features
  and no dimensionality reduction has been applied — hyperparameter guidance differs
  (e.g., lambda1=0.001, lambda2=5, lambda3=5 rather than 1/2/1)
- When a pure-Python kBET implementation is required — kBET is R-only and must be
  reached via rpy2
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3524
tools:
- name: BEENE
  role: Trains the deep nonlinear autoencoder (get_hybrid_model_1, train_model, get_beene_embeddings)
    that produces the nonlinear embedding and iLISI evaluation (evaluate_batch_iLisi)
  repo: https://github.com/ashiq24/BEENE
- name: scikit-learn
  role: Provides OneHotEncoder for batch label encoding, train_test_split, and the
    PCA embedding used as the linear baseline
- name: numpy
  role: Array handling and np.savetxt storage of learned BEENE embeddings
- name: rpy2
  role: Transfers the learned nonlinear embedding from Python to the R environment
    for kBET calculation
- name: kBET
  role: R package computing average k-nearest-neighbour batch rejection rates via
    Pearson's chi-square test on the embeddings
  repo: https://github.com/theislab/kBET
- name: LISI
  role: Source of the iLISI batch mixing metric computed via beene_model.evaluate_batch_iLisi
  repo: https://github.com/immunogenomics/LISI
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad479
    title: 'BEENE: deep learning-based nonlinear embedding improves batch effect estimation'
  source_content_hash: sha256:d0705f9182c508e58d409f2d034765d0e2cab46fe994d2da84ad83834d4cd21f
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# pca-vs-nonlinear-embedding-batch-benchmarking

## Summary

Benchmarks batch-effect quantification on a PCA embedding versus a BEENE nonlinear autoencoder embedding, evaluating kBET rejection rates and iLISI before and after correction (e.g., ComBat, Harmony) to test whether a nonlinear embedding reflects batch effects more sensitively than PC space.

## When to use

Use when quantifying batch effects in single-cell RNA-seq data where PCA-space metrics may miss nonlinear batch structure — e.g., comparing uncorrected vs. ComBat/Harmony-corrected data with kBET and iLISI.

## When NOT to use

- When the goal is batch correction itself rather than quantifying/estimating batch effect extent — BEENE estimates effects, it does not remove them
- When the embedding input is a high-dimensional dataset with thousands of features and no dimensionality reduction has been applied — hyperparameter guidance differs (e.g., lambda1=0.001, lambda2=5, lambda3=5 rather than 1/2/1)
- When a pure-Python kBET implementation is required — kBET is R-only and must be reached via rpy2

## Inputs

- Processed single-cell gene-expression matrix (cells x genes) with batch labels, e.g., the three Chromium 10X PBMC datasets (3pV1, 3pV2, 5p) from immunogenomics/harmony2019 data/figure4
- Batch variable per cell (3 categories -> one-hot dense matrix via scikit-learn OneHotEncoder)
- Biological variable per cell (two classes -> 1D 0/1 array)
- Corrected and uncorrected versions of each condition (e.g., ComBat-corrected, Harmony-corrected)

## Outputs

- BEENE embeddings per condition saved via np.savetxt to 'embedding.txt'
- PCA embeddings per condition (scikit-learn)
- iLISI values on the test set for PCA and BEENE embeddings per correction state
- kBET average rejection rates for PCA and BEENE embeddings per correction state (computed in R via rpy2)
- Comparison table batch_metrics.csv (PCA vs BEENE x uncorrected/ComBat/Harmony)

## How to apply

Load the multi-batch dataset (e.g., three Chromium 10X PBMC experiments 3pV1/3pV2/5p from the harmony2019 figure4 deposit) and prepare uncorrected and corrected versions. One-hot encode the batch labels as a dense matrix when there are >2 categories (scikit-learn OneHotEncoder); a 1D 0/1 array suffices for two-class biological variables. Compute a scikit-learn PCA embedding as the linear baseline, then train the BEENE autoencoder per condition with beene_model.get_hybrid_model_1 (e.g., hidden layers [50,20], embedding dimension 5, reconstruction_weight=1, batch_weight=2, bio_weight=1) using a train/validation/test split, saving embeddings with np.savetxt to 'embedding.txt'. Evaluate iLISI on the test set with evaluate_batch_iLisi (seed=20) and transfer each learned embedding to R (>=4.1.0) via rpy2 to run kBET for rejection rates. Assemble kBET rejection rates and median iLISI per embedding per correction state into a comparison table (batch_metrics.csv).

## Related tools

- **BEENE** (Trains the deep nonlinear autoencoder (get_hybrid_model_1, train_model, get_beene_embeddings) that produces the nonlinear embedding and iLISI evaluation (evaluate_batch_iLisi)) — https://github.com/ashiq24/BEENE
- **scikit-learn** (Provides OneHotEncoder for batch label encoding, train_test_split, and the PCA embedding used as the linear baseline)
- **numpy** (Array handling and np.savetxt storage of learned BEENE embeddings)
- **rpy2** (Transfers the learned nonlinear embedding from Python to the R environment for kBET calculation)
- **kBET** (R package computing average k-nearest-neighbour batch rejection rates via Pearson's chi-square test on the embeddings) — https://github.com/theislab/kBET
- **LISI** (Source of the iLISI batch mixing metric computed via beene_model.evaluate_batch_iLisi) — https://github.com/immunogenomics/LISI

## Examples

```
my_model.get_hybrid_model_1(100,[50,20],5,3,2,1,2,1); my_model.train_model(X_train,Y_Platform_train,Y_ER_train,X_val,Y_Platform_val,Y_ER_val,100); np.savetxt('embedding.txt', my_model.get_beene_embeddings(X_test), fmt='%f')
```

## Evaluation signals

- Reproduced pattern should match Fig. 4: kBET rejection rate ~1.00 for uncorrected and ComBat-corrected data (PCA ComBat ~0.96), dropping to ~0.80 for BEENE on Harmony-corrected PBMC data
- Median iLISI should rise from ~1 (uncorrected) to ~2.5 on BEENE embeddings after Harmony correction
- Batch labels with >2 categories must be one-hot encoded as a dense matrix; two-class variables must be a 1D 0/1 array, otherwise evaluate_batch_iLisi input contract is violated
- Model training should use a train/validation/test split with the best model selected on validation and metrics computed on the test set (seed=20 for iLISI)
- BEENE hyperparameters follow Table 1 guidance for reduced-dimension data (reconstruction_weight=1, batch_weight=2, bio_weight=1)

## Limitations

- kBET has no Python implementation and is available only in R, requiring rpy2 transfer (Linux recommended for rpy2 >= 3.4.0)
- If kBET returns an average rejection rate of 1 for batch-corrected data, additional measures such as average silhouette width or PCA-based batch-effect measures may be needed to explore the degree of batch effect
- BEENE hyperparameter weights (lambda1, lambda2, lambda3) are data- and application-dependent; Table 1 guidelines apply only under the stated dimensionality conditions
- rpy2 usage requires R >= 4.1.0 and Python >= 3.7.0

## Evidence

- [results] On the Harmony-corrected PBMC data, the kBET rejection rate in the BEENE embedding drops to 0.80, whereas it was close to 1.00 for both the uncorrected and ComBat-corrected data: [evidence span withheld — anchor status: not_found]
- [readme] get_hybrid_model_1 signature: "get_hybrid_model_1(number_of_genes, hidden_layer_dimensions, embedding_dimension,\
                   number_of_batches, number_of_biological_variables, reconstruction_weight,\"
- [readme] kBET rejection rate meaning: [evidence span withheld — anchor status: external_doc]
- [readme] One-hot requirement: "For more than two categories, it must be one hot representation of 
                batch labels for each of the samples in the data and must be a dense matrix."
- [supplementary] Batch metrics comparison: [evidence span withheld — anchor status: not_found]
- [abstract] BEENE rationale: "BEENE simultaneously learns the batch and biological variables from RNA-seq data, resulting in an embedding that is more robust and sensitive than PCA embedding in terms of detecting and quantifying"
