---
name: combat-batch-effect-correction
description: Use when you have an expression matrix with technical batch labels (e.g.,
  two mESC replicates) that has been ComBat-corrected, and you need to detect residual
  batch effects and compare sensitivity of PCA vs BEENE embedding via kBET and iLISI.
when_to_use_negative:
- Data has no batch labels at all — ComBat cannot estimate adjustment parameters and
  kBET/iLISI cannot score batch mixing without a batch variable
- The goal is to remove/integrate batches rather than detect and quantify (residual)
  batch effects — the BEENE paper positions the method for estimation of batch effects,
  not correction
- Data is already known to be fully batch-mixed (e.g., ComBat correction left no residual
  effect) — kBET is very sensitive and a rejection rate near 0 in PCA space leaves
  little for the nonlinear embedding comparison to add
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BEENE
  role: Deep nonlinear auto-encoder that produces the alternative lower-dimensional
    embedding used to probe residual batch effects in the ComBat-corrected mESC data
  repo: https://github.com/ashiq24/BEENE
- name: scikit-learn
  role: Provides the PCA embedding of the ComBat-corrected data used as the linear
    baseline for comparison
- name: rpy2
  role: Transfers the Python-computed embeddings into the R environment so the kBET
    R package can be run
- name: kBET
  role: R package that scores each embedding by average chi-squared rejection rate
    of batch mixing in k-nearest-neighbourhoods
  repo: https://github.com/theislab/kBET/
- name: LISI
  role: Source of the iLISI metric computed on PCA and BEENE embeddings via evaluate_batch_iLisi
  repo: https://github.com/immunogenomics/LISI
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad479
    title: 'BEENE: deep learning-based nonlinear embedding improves batch effect estimation'
  source_content_hash: sha256:7c23ddb5ee4d8260323ed42ef5d6fde00dd11a91053ccb6f4ce512afb970f2a4
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# combat-batch-effect-correction

## Summary

Apply ComBat batch correction to a replicate-level RNA-seq matrix (e.g., the GSE65525 mESC technical-replicate data), then quantify residual (leftover) batch effects by comparing kBET rejection rates and iLISI in PCA space versus the BEENE nonlinear embedding, which the BEENE paper shows detects leftover batch effects more sensitively than PCA.

## When to use

Use when you have an expression matrix with technical batch labels (e.g., two mESC replicates) that has been ComBat-corrected, and you need to detect residual batch effects and compare sensitivity of PCA vs BEENE embedding via kBET and iLISI.

## When NOT to use

- Data has no batch labels at all — ComBat cannot estimate adjustment parameters and kBET/iLISI cannot score batch mixing without a batch variable
- The goal is to remove/integrate batches rather than detect and quantify (residual) batch effects — the BEENE paper positions the method for estimation of batch effects, not correction
- Data is already known to be fully batch-mixed (e.g., ComBat correction left no residual effect) — kBET is very sensitive and a rejection rate near 0 in PCA space leaves little for the nonlinear embedding comparison to add

## Inputs

- ComBat-corrected (or raw + batch-label) gene expression matrix as a 2D numpy array, one row per sample and one column per gene (mESC technical replicate data, GSE65525)
- Batch variable per sample: 1D array of zeros/ones for two categories, or a dense one-hot matrix for >2 categories
- Biological variable per sample (same encoding rules; None if absent)
- Fixed integer random seed for the train/validation/test split

## Outputs

- PCA embedding of the ComBat-corrected expression data
- BEENE embedding (e.g., saved via np.savetxt('embedding.txt', test_embedding, fmt='%f'))
- Per-sample iLISI values for each embedding (from evaluate_batch_iLisi)
- kBET average rejection rate for each embedding (from the R kBET package, called via rpy2)
- Comparison table of kBET and iLISI values (BEENE kBET 0.58 vs PCA 0.33/0.20; iLISI 1.57 BEENE vs ~1.76 PCA)

## How to apply

Load the replicate expression matrix and its batch labels (e.g., two technical replicates of mouse embryogenesis cells, GSE65525) into 2D numpy arrays and apply ComBat batch correction to obtain the corrected matrix. Compute a PCA embedding of the corrected data with scikit-learn as the linear baseline, then build the BEENE model with get_hybrid_model_1 (article example: get_hybrid_model_1(100,[50,20],5,3,2,1,2,1), i.e., 100 genes, hidden layers [50,20], 5-dimensional embedding, reconstruction/batch/bio weights 1/2/1) and train it for 300 epochs with train_model on train/validation splits. Compute iLISI with evaluate_batch_iLisi on both embeddings using a fixed random seed (with a 1D 0/1 batch array for two categories), and transfer each embedding to R via rpy2 to run the kBET package, which returns an average chi-squared rejection rate. Correct application is judged by reproducing the reported comparison on ComBat-corrected mESC data: BEENE kBET rejection 0.58 vs PCA 0.33 (full) and 0.20 (test), and iLISI of 1.57 in the BEENE embedding vs ~1.76 in PCA space — the higher BEENE kBET rate indicates the nonlinear embedding exposes residual batch structure that PCA under-detects.

## Related tools

- **BEENE** (Deep nonlinear auto-encoder that produces the alternative lower-dimensional embedding used to probe residual batch effects in the ComBat-corrected mESC data) — https://github.com/ashiq24/BEENE
- **scikit-learn** (Provides the PCA embedding of the ComBat-corrected data used as the linear baseline for comparison)
- **rpy2** (Transfers the Python-computed embeddings into the R environment so the kBET R package can be run)
- **kBET** (R package that scores each embedding by average chi-squared rejection rate of batch mixing in k-nearest-neighbourhoods) — https://github.com/theislab/kBET/
- **LISI** (Source of the iLISI metric computed on PCA and BEENE embeddings via evaluate_batch_iLisi) — https://github.com/immunogenomics/LISI

## Examples

```
my_model.get_hybrid_model_1(100,[50,20],5,3,2,1,2,1); my_model.train_model(X_train,Y_Platform_train,Y_ER_train,X_val,Y_Platform_val,Y_ER_val,100)
```

## Evaluation signals

- Reproduce the reported kBET rejection rates on ComBat-corrected mESC data: BEENE embedding 0.58 vs PCA 0.33 (full dataset) and 0.20 (test dataset)
- Reproduce iLISI of 1.57 in the BEENE embedding vs ~1.76 in PCA space
- Batch variable encoding invariant: 1D 0/1 array for two categories; dense one-hot matrix for more than two categories
- Model trained for the documented 300 epochs with train/validation splits, with model selection on the validation set
- Sanity check per kBET README: lower rejection rate means less batch bias, so ComBat correction should lower rejection relative to uncorrected data, while BEENE should still flag residual effect (higher rate than PCA)

## Limitations

- Existing batch effect estimation techniques that depend on the local distribution of cells in PC space may fail to capture sophisticated non-linear batch effects — a PCA comparison can under-detect residual effects
- kBET returns a binary result per tested sample and averages a rejection rate; if it returns an average rejection rate of 1 on batch-corrected data, the README advises additionally computing average silhouette width and PCA-based batch-effect measures
- rpy2-based kBET transfer is recommended on Linux; R >= 4.1.0 and Python >= 3.7.0 are required
- BEENE hyperparameters (lambda1 reconstruction weight, lambda2 batch weight, lambda3 bio weight) are data-dependent: use 1/2/1 when features were reduced to hundreds, but decrease lambda1 (e.g., by a factor of 1/1000, i.e., 0.001) when thousands of features are used without dimensionality reduction; higher batch_weight makes BEENE more sensitive to batch effect

## Evidence

- [other] Apply ComBat batch correction to the expression matrix.: [evidence span withheld — anchor status: not_found]
- [results] On ComBat-corrected mESC data, BEENE achieved a higher kBET rejection rate (0.58) than the PCA space (0.33 and 0.20 on the full and test datasets).: [evidence span withheld — anchor status: not_found]
- [readme] The lower the test result, the less bias is introduced by the batch effect.: [evidence span withheld — anchor status: external_doc]
- [intro] BEENE is the first known method to explicitly consider and model the non-linearity of batch effects for effective batch effect detection and estimation.: "BEENE is the first known method to explicitly consider and model the non-linearity of batch effects for effective batch effect detection and estimation."
- [methods] lisi_values = my_model.evaluate_batch_iLisi(Xt,bt,yt,20): "lisi_values = my_model.evaluate_batch_iLisi(Xt,bt,yt,20)"
