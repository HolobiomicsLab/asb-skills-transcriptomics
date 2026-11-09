---
name: scrna-batch-mixing-ilisi-quantification
description: Use when comparing batch mixing of two embedding spaces (e.g., BEENE
  vs PCA) on scRNA-seq data with batch labels and optionally biological labels, to
  test which space more sensitively detects batch effects via iLISI/kBET.
when_to_use_negative:
- The input embedding already comes from a batch-correction/integration method and
  the goal is post-correction mixing assessment — kBET's README advises that if rejection
  rate is 1 on corrected data you should additionally use silhouette width and PCA-based
  measures to gauge severity, not iLISI alone.
- The batch variable is undefined or the data has no replicate/study structure — iLISI
  measures mixing across a categorical batch variable and cannot be computed without
  one-hot-encoded batch labels.
- You only have Python and cannot install R (>= 4.1.0) with rpy2 (>= 3.4.0) — kBET
  has no Python implementation and is available only in R, so the kBET cross-check
  is not feasible.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: BEENE
  role: 'Provides the nonlinear auto-encoder embedding (beene_model: get_hybrid_model_1,
    train_model, get_beene_embeddings, evaluate_batch_iLisi) on which iLISI is computed'
  repo: https://github.com/ashiq24/BEENE
- name: scikit-learn
  role: OneHotEncoder for dense batch-label encoding and PCA for the top-50 linear
    baseline embedding
- name: LISI
  role: R package computing the Local Inverse Simpson's Index used as the batch-mixing
    metric
  repo: https://github.com/immunogenomics/LISI
- name: kBET
  role: R-only package (run via rpy2) producing Pearson chi-squared-based average
    rejection rates on each embedding for full and test data
  repo: https://github.com/theislab/kBET/
- name: rpy2
  role: Transfers PCA-50 and BEENE embeddings from Python to the R environment for
    kBET calculation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad479
    title: 'BEENE: deep learning-based nonlinear embedding improves batch effect estimation'
  source_content_hash: sha256:e3539a32dcb90606d516592df8bf028f3c5387f78ecd41c0de74090fb512a8d4
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
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# scrna-batch-mixing-ilisi-quantification

## Summary

Quantify cell-type/batch mixing in single-cell RNA-seq embeddings by computing iLISI (inverse Simpson's index) per cell on a chosen embedding space (BEENE nonlinear embedding or PCA-50 baseline), optionally paired with R kBET rejection rates to cross-validate batch-effect detection.

## When to use

Use when comparing batch mixing of two embedding spaces (e.g., BEENE vs PCA) on scRNA-seq data with batch labels and optionally biological labels, to test which space more sensitively detects batch effects via iLISI/kBET.

## When NOT to use

- The input embedding already comes from a batch-correction/integration method and the goal is post-correction mixing assessment — kBET's README advises that if rejection rate is 1 on corrected data you should additionally use silhouette width and PCA-based measures to gauge severity, not iLISI alone.
- The batch variable is undefined or the data has no replicate/study structure — iLISI measures mixing across a categorical batch variable and cannot be computed without one-hot-encoded batch labels.
- You only have Python and cannot install R (>= 4.1.0) with rpy2 (>= 3.4.0) — kBET has no Python implementation and is available only in R, so the kBET cross-check is not feasible.

## Inputs

- Raw scRNA-seq expression matrix as 2D numpy array (rows = samples, columns = genes), e.g., mESC day-0 inDrop technical replicate data (GSE65525)
- Batch variable labels as numpy array (dense one-hot matrix for >2 categories; 1D 0/1 array for 2 categories)
- Biological variable labels as numpy array (same encoding convention; None if absent)
- PCA-50 embedding and BEENE embedding for the same cells
- Random seed integer for the train-validation-test split (e.g., 20)

## Outputs

- Per-sample iLISI values for the test set on each embedding
- Median iLISI (np.median(lisi_values))
- kBET average rejection rates per embedding on full data and test data
- Results table, e.g., kBET_iLISI_mESC_replicates.csv, with PCA-50 vs BEENE metrics

## How to apply

Prepare the expression matrix (samples x genes) with batch variables one-hot encoded via sklearn's OneHotEncoder into a dense matrix (a 1D 0/1 array suffices for two categories), plus biological labels if available. Compute the linear baseline as the top-50 principal components with scikit-learn PCA, and obtain the nonlinear alternative from the beene_model class: build with get_hybrid_model_1, train with train_model, and extract embeddings via get_beene_embeddings, using a fixed random seed (e.g., 20) for the train-validation-test split. Call evaluate_batch_iLisi(data, batch_var, bio_var, seed), which trains on the training set, selects the best model on the validation set, and returns per-sample iLISI on the randomly chosen test set; report the median via np.median(lisi_values). For the kBET cross-check, transfer both PCA-50 and BEENE embeddings from Python to R (>= 4.1.0) via rpy2 (>= 3.4.0) and run the R-only kBET package, which computes Pearson's chi-squared tests on the batch label distribution in k-nearest-neighbourhoods of 10% of samples, reporting an average rejection rate (lower = less batch bias). Write per-sample iLISI, median iLISI, and kBET rejection rates to a CSV (e.g., kBET_iLISI_mESC_replicates.csv) and compare embeddings.

## Related tools

- **BEENE** (Provides the nonlinear auto-encoder embedding (beene_model: get_hybrid_model_1, train_model, get_beene_embeddings, evaluate_batch_iLisi) on which iLISI is computed) — https://github.com/ashiq24/BEENE
- **scikit-learn** (OneHotEncoder for dense batch-label encoding and PCA for the top-50 linear baseline embedding)
- **LISI** (R package computing the Local Inverse Simpson's Index used as the batch-mixing metric) — https://github.com/immunogenomics/LISI
- **kBET** (R-only package (run via rpy2) producing Pearson chi-squared-based average rejection rates on each embedding for full and test data) — https://github.com/theislab/kBET/
- **rpy2** (Transfers PCA-50 and BEENE embeddings from Python to the R environment for kBET calculation)

## Examples

```
lisi_values = my_model.evaluate_batch_iLisi(Xt, bt, yt, 20); print(np.median(lisi_values))
```

## Evaluation signals

- Per-sample iLISI values are returned for every sample in the randomly chosen test set; median iLISI is printed via np.median(lisi_values) and written to the CSV.
- On uncorrected mESC day-0 replicates, kBET with the BEENE embedding should reach a rejection rate of 1.00 while PCA-50 gives ~0.80 (full data) and ~0.48 (test data); reproducing these values validates the run.
- knn/neighbourhood checks: kBET tests 10% of samples against the local batch label distribution with a fixed neighbourhood size k; higher rejection rate indicates stronger detected batch effect.
- Embedding files: BEENE test embeddings are saved with np.savetxt('embedding.txt', test_embedding, fmt='%f') and should have the configured embedding_dimension columns (e.g., 50 in get_hybrid_model_1(100,[50,20],5,3,2,1,2,1)).
- Both PCA-50 and BEENE embeddings must be evaluated on the same cells and same train-validation-test split (fixed seed) for a fair comparison.

## Limitations

- kBET is only available in R and must be called via rpy2 (>= 3.4.0) with R >= 4.1.0; Linux is recommended for seamless operation.
- kBET is very sensitive to any kind of bias: if it returns an average rejection rate of 1 on batch-corrected data, you should additionally compute average silhouette width and PCA-based batch-effect measures.
- BEENE hyperparameters are data-dependent: with batch_weight higher than reconstruction_weight/bio_weight, embeddings cluster more tightly by batch (more sensitive to batch effect); the guideline table recommends lambda1=1, lambda2=2, lambda3=1 when features are reduced to hundreds, and lambda1 reduced (e.g., by 1/1000, e.g., 0.001) for thousands of features.
- evaluate_batch_iLisi results depend on the random seed of the train-validation-test split (e.g., seed 20), so values are split-dependent.

## Evidence

- [methods] lisi_values = my_model.evaluate_batch_iLisi(Xt,bt,yt,20): "lisi_values = my_model.evaluate_batch_iLisi(Xt,bt,yt,20)"
- [readme] Compute the iLISI metric on the data. The data is split into test, training, and validation set.: "Compute the iLISI metric on the data. The data is split into test, training, and validation set."
- [readme] the result of kBET is the average test rejection rate. The lower the test result, the less bias is introduced by the batch effect.: [evidence span withheld — anchor status: external_doc]
- [methods] To calculate kBET using non-linear embedding, the learned embedding must be transferred to R environment for calculation.: "To calculate kBET using non-linear embedding, the learned embedding must be transferred to R environment for calculation."
- [abstract] BEENE simultaneously learns the batch and biological variables from RNA-seq data, resulting in an embedding that is more robust and sensitive than PCA: "BEENE simultaneously learns the batch and biological variables from RNA-seq data, resulting in an embedding that is more robust and sensitive than PCA embedding in terms of detecting and quantifying"
