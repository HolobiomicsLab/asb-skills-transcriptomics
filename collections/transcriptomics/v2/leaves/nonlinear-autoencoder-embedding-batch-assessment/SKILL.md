---
name: nonlinear-autoencoder-embedding-batch-assessment
description: Use when batch effects in RNA-seq/single-cell data must be detected and
  quantified (kBET/iLISI) and PCA-space estimates may miss non-linear batch effects,
  e.g. multi-replicate inDrop mESC data.
when_to_use_negative:
- When the goal is batch correction/integration rather than detection and estimation
  — BEENE quantifies batch effects; it does not return corrected expression data
- When input is already an embedding or batch-corrected matrix rather than the raw
  samples-x-genes expression the autoencoder is trained on
- When the batch variable is unlabeled — evaluate_batch_iLisi and kBET both require
  batch labels (and bio_var labels) to score mixing
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BEENE
  role: beene_model class providing get_hybrid_model_1, train_model, get_beene_embeddings,
    and evaluate_batch_iLisi to train the autoencoder, extract embeddings, and compute
    iLISI
  repo: https://github.com/ashiq24/BEENE
- name: scikit-learn
  role: OneHotEncoder to one-hot encode multi-category batch variables and PCA to
    compute the top-50 linear-embedding baseline space
- name: rpy2
  role: Transfers the PCA-50 and BEENE embeddings from Python to the R environment
    for kBET calculation (requires rpy2 >= 3.4.0)
- name: kBET
  role: R-only k-nearest neighbour batch effect test returning average rejection rates
    on each embedding (full data and test data)
  repo: https://github.com/theislab/kBET/
- name: LISI
  role: Source of the iLISI metric (Local Inverse Simpson's Index) computed per sample
    on the BEENE embedding
  repo: https://github.com/immunogenomics/LISI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad479
    title: 'BEENE: deep learning-based nonlinear embedding improves batch effect estimation'
  source_content_hash: sha256:7d54d384edaf82b15c68ff11b83731e82c0b40343fe0553ae0632854a0290087
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# nonlinear-autoencoder-embedding-batch-assessment

## Summary

Uses the BEENE deep nonlinear auto-encoder to learn a lower-dimensional embedding from RNA-seq data (learning batch and biological variables simultaneously), then quantifies batch effects with iLISI (Python) and kBET rejection rates (R via rpy2), which is more sensitive than estimating effects in PCA space.

## When to use

Use when batch effects in RNA-seq/single-cell data must be detected and quantified (kBET/iLISI) and PCA-space estimates may miss non-linear batch effects, e.g. multi-replicate inDrop mESC data.

## When NOT to use

- When the goal is batch correction/integration rather than detection and estimation — BEENE quantifies batch effects; it does not return corrected expression data
- When input is already an embedding or batch-corrected matrix rather than the raw samples-x-genes expression the autoencoder is trained on
- When the batch variable is unlabeled — evaluate_batch_iLisi and kBET both require batch labels (and bio_var labels) to score mixing

## Inputs

- Raw RNA-seq expression matrix as 2D numpy array (samples x genes), e.g. mESC day-0 inDrop technical replicate matrix (also GSE65525 mESC, PBMC 3-experiment, pancreatic islet 5-study processed data)
- Batch variable labels: 1D 0/1 numpy array for two categories or dense one-hot matrix for >2 categories
- Biological variable labels (1D 0/1 or dense one-hot), or None if absent
- Integer random seed for the train-validation-test split (e.g. seed 20)
- A configured beene_model and R environment (R >= 4.1.0) with the kBET package for the rejection-rate step

## Outputs

- BEENE embedding matrix (one row per sample, embedding_dimension columns), optionally saved with np.savetxt('embedding.txt', test_embedding, fmt='%f')
- Per-sample iLISI values for the randomly chosen test set from evaluate_batch_iLisi, plus the median iLISI via np.median(lisi_values)
- kBET average rejection rates for each embedding (e.g. BEENE 1.00, PCA-50 0.80 full / 0.48 test) written to a results file such as kBET_iLISI_mESC_replicates.csv

## How to apply

Build the model with beene_model.get_hybrid_model_1 (article example: get_hybrid_model_1(100,[50,20],5,3,2,1,2,1) — genes, hidden dims [50,20], embedding dim, number of batches, number of biological variables, and reconstruction/batch/bio weights), one-hot encoding multi-category batch labels with sklearn's OneHotEncoder into dense matrices (a 1D 0/1 array suffices for two categories). Split data into training-validation-test sets with a fixed seed (e.g. 20), train with train_model (e.g. 300 epochs, batch_size 32), select the best model on the validation set, and obtain embeddings via get_beene_embeddings(data). Compute per-sample iLISI with evaluate_batch_iLisi(data, batch_var, bio_var, seed) on the test set and report np.median(lisi_values). Choose weights per README Table 1: λ1=λ3=1 and λ2=2 when features were reduced to hundreds; λ1=0.001, λ2=5, λ3=5 for thousands of raw features (set islarge=True for high-dimensional data, which raises input-layer dropout). For kBET, which has no Python implementation, transfer the top-50 scikit-learn PCA embedding and the BEENE embedding to R (>= 4.1.0) via rpy2 (>= 3.4.0) and compare rejection rates, e.g. BEENE 1.00 vs PCA-50 0.80 (full) / 0.48 (test) on uncorrected mESC day-0 replicates, writing results to kBET_iLISI_mESC_replicates.csv.

## Related tools

- **BEENE** (beene_model class providing get_hybrid_model_1, train_model, get_beene_embeddings, and evaluate_batch_iLisi to train the autoencoder, extract embeddings, and compute iLISI) — https://github.com/ashiq24/BEENE
- **scikit-learn** (OneHotEncoder to one-hot encode multi-category batch variables and PCA to compute the top-50 linear-embedding baseline space)
- **rpy2** (Transfers the PCA-50 and BEENE embeddings from Python to the R environment for kBET calculation (requires rpy2 >= 3.4.0))
- **kBET** (R-only k-nearest neighbour batch effect test returning average rejection rates on each embedding (full data and test data)) — https://github.com/theislab/kBET/
- **LISI** (Source of the iLISI metric (Local Inverse Simpson's Index) computed per sample on the BEENE embedding) — https://github.com/immunogenomics/LISI

## Examples

```
lisi_values = my_model.evaluate_batch_iLisi(Xt, bt, yt, 20)
```

## Evaluation signals

- Reproduces the reported mESC day-0 comparison: BEENE kBET rejection rate 1.00 vs PCA-50 rejection rates 0.80 (full data) and 0.48 (test data), with similar iLISI trends favoring BEENE
- evaluate_batch_iLisi returns iLISI values only for the samples of the randomly chosen (seeded) test set, not the full data; median is reported with print(np.median(lisi_values))
- batch_var/bio_var format check passes: 1D 0/1 array for two categories, dense one-hot matrix for >2 categories
- Embedding round-trip to R succeeds (rpy2 >= 3.4.0, R >= 4.1.0) and kBET returns rejection rates within [0, 1] for both PCA and BEENE spaces
- Per README Table 1, chosen weights match the feature regime: λ1=1, λ2=2 (and λ3=1) for hundreds of features after dimensionality reduction; λ1=0.001, λ2=5, λ3=5 for thousands of features

## Limitations

- kBET has no Python implementation and is available only in R; the rpy2 bridge is recommended on Linux systems for seamless working
- Hyperparameter choice (λ1 reconstruction_weight, λ2 batch_weight, λ3 bio_weight) is data-dependent: higher batch_weight makes BEENE more sensitive to batch effects, and the guidance differs for features in the hundreds vs thousands
- For high-dimensional datasets islarge=True must be set so the model applies a higher dropout rate in the input layer
- If kBET returns an average rejection rate of 1 on batch-corrected data, the README advises additionally computing average silhouette width and PCA-based batch-effect measures to gauge the degree of the effect

## Evidence

- [abstract] BEENE deep nonlinear autoencoder embedding + kBET/iLISI batch assessment: "We present Batch Effect Estimation using Nonlinear Embedding (BEENE), a deep nonlinear auto-encoder network which is specially tailored to generate an alternative lower dimensional embedding"
- [intro] Rationale for non-linear modeling: "BEENE is the first known method to explicitly consider and model the non-linearity of batch effects for effective batch effect detection and estimation."
- [methods] Model construction call: "my_model.get_hybrid_model_1(100,[50,20],5,3,2,1,2,1)"
- [methods] iLISI computation on embedding: "lisi_values = my_model.evaluate_batch_iLisi(Xt,bt,yt,20)"
- [methods] rpy2 bridge to R kBET: "To calculate kBET using non-linear embedding, the learned embedding must be transferred to R environment for calculation."
- [readme] iLISI output semantics: "returns: iLISI values of each of the samples in the randomly chosen test set using BEENE embedding"
- [readme] Hyperparameter guidance: "We recommend to use  $\lambda_1 = \lambda_3 = 1$ and $\lambda_2 = 2$ when a dimensionality reduction technique has been used to reduce the number of features to hundreds."
- [readme] kBET rejection-rate interpretation caveat: [evidence span withheld — anchor status: external_doc]
