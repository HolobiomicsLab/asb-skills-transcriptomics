---
name: pca-linear-embedding-baseline-construction
description: Use when you need a standard linear embedding baseline in PC space to
  benchmark a learned nonlinear embedding's sensitivity for batch effect estimation
  metrics like kBET or iLISI on single-cell RNA-seq data.
when_to_use_negative:
- The input is already a batch-corrected or feature-reduced embedding and the goal
  is integration-quality assessment rather than building a linear baseline — kBET
  rejection rate of 1 on corrected data calls for silhouette-width or PCA-based batch-effect
  measures instead.
- The analysis goal is batch correction itself (e.g., Harmony) rather than batch effect
  detection/estimation via embedding comparison.
- The data has only a handful of features after upstream dimensionality reduction
  to the embedding dimension, making PCA-50 redundant.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scikit-learn
  role: Provides the PCA implementation used to compute the top-50 principal components
    of the raw data as the linear-embedding baseline, plus OneHotEncoder for batch/bio
    label encoding
- name: numpy
  role: Holds the 2D samples-x-genes expression array and the resulting PCA-50 embedding
    matrix
- name: rpy2
  role: Transfers the PCA-50 (and BEENE) embeddings from Python to R to run the R-only
    kBET test
- name: kBET
  role: R package that computes the average test rejection rate on the PCA space via
    Pearson's chi-square test on k-nearest-neighbour batch label distributions
  repo: https://github.com/theislab/kBET/
- name: LISI
  role: Computes iLISI (Local Inverse Simpson's Index) on the PCA space as a second
    baseline batch-mixing metric
  repo: https://github.com/immunogenomics/LISI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad479
    title: 'BEENE: deep learning-based nonlinear embedding improves batch effect estimation'
  source_content_hash: sha256:147d0d9174c78ecc21666b8d6709e960bc37f407d2384d97f21ba70b616a7df5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/kBET
  tools:
  - LISI
  - R
  - kBET
  - numpy
  - rpy2
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# pca-linear-embedding-baseline-construction

## Summary

Construct a top-50 principal component (PCA) space of the raw single-cell RNA-seq expression matrix with scikit-learn, serving as the linear-embedding baseline against which a nonlinear embedding (e.g., BEENE) is compared for batch effect detection with kBET and iLISI.

## When to use

Use when you need a standard linear embedding baseline in PC space to benchmark a learned nonlinear embedding's sensitivity for batch effect estimation metrics like kBET or iLISI on single-cell RNA-seq data.

## When NOT to use

- The input is already a batch-corrected or feature-reduced embedding and the goal is integration-quality assessment rather than building a linear baseline — kBET rejection rate of 1 on corrected data calls for silhouette-width or PCA-based batch-effect measures instead.
- The analysis goal is batch correction itself (e.g., Harmony) rather than batch effect detection/estimation via embedding comparison.
- The data has only a handful of features after upstream dimensionality reduction to the embedding dimension, making PCA-50 redundant.

## Inputs

- Raw (uncorrected) samples-x-genes single-cell RNA-seq expression matrix as a 2D numpy array (e.g., mESC day-0 inDrop technical replicate data from GSE65525)
- Batch/replicate labels per sample (1D 0/1 array for two categories; one-hot dense matrix for >2 categories)
- Biological labels per sample (same encoding rules as batch labels)

## Outputs

- Top-50 PCA embedding (n_cells x 50 matrix) of the raw expression data
- kBET average rejection rates computed on the PCA space (full data and test split), e.g., 0.80 (full) / 0.48 (test) on mESC replicates
- Per-sample iLISI values and median iLISI on the PCA space, exported with the nonlinear-embedding results to kBET_iLISI_mESC_replicates.csv

## How to apply

Load the raw samples-x-genes expression matrix (e.g., the mESC day-0 inDrop technical replicate data from GSE65525) and use scikit-learn PCA to compute the top-50 principal components of the uncorrected data as the linear-embedding baseline space. Keep this baseline on exactly the same cells and the same train-validation-test split protocol (fixed random seed, e.g., seed 20) as the nonlinear embedding so the comparison is apples-to-apples. One-hot encode multi-category batch variables with sklearn's OneHotEncoder (a 1D 0/1 array suffices for two categories such as two technical replicates) before feeding labels to kBET/iLISI. Transfer both the PCA-50 and nonlinear embeddings from Python to R (R >= 4.1.0) via rpy2 (>= 3.4.0) and run the R-only kBET package to compute average rejection rates for each embedding on the full data and the test set. The rationale is that existing batch effect estimation techniques depend on the local distribution of cells in PC space, which may fail to capture sophisticated non-linear batch effects — quantifying how much the nonlinear embedding changes the rejection rate relative to PCA is the whole point of the baseline.

## Related tools

- **scikit-learn** (Provides the PCA implementation used to compute the top-50 principal components of the raw data as the linear-embedding baseline, plus OneHotEncoder for batch/bio label encoding)
- **numpy** (Holds the 2D samples-x-genes expression array and the resulting PCA-50 embedding matrix)
- **rpy2** (Transfers the PCA-50 (and BEENE) embeddings from Python to R to run the R-only kBET test)
- **kBET** (R package that computes the average test rejection rate on the PCA space via Pearson's chi-square test on k-nearest-neighbour batch label distributions) — https://github.com/theislab/kBET/
- **LISI** (Computes iLISI (Local Inverse Simpson's Index) on the PCA space as a second baseline batch-mixing metric) — https://github.com/immunogenomics/LISI

## Examples

```
from sklearn.decomposition import PCA; pca50 = PCA(n_components=50).fit_transform(raw_counts)
```

## Evaluation signals

- On the uncorrected mESC day-0 inDrop two-replicate data, kBET on the PCA-50 space should reproduce reported rejection rates of 0.80 on the full data and 0.48 on the test set
- The PCA rejection rates should be lower than the BEENE embedding rejection rate (1.00) for this dataset, confirming the baseline is less sensitive here
- The PCA embedding must be computed on the same cells and same fixed-seed train-validation-test split (e.g., seed 20) as the nonlinear embedding for a valid comparison
- Output CSV (kBET_iLISI_mESC_replicates.csv) must contain per-sample iLISI values, median iLISI, and kBET rejection rates for both PCA-50 and BEENE spaces
- kBET should return a binary test result per tested sample and an average rejection rate in [0, 1]; values outside this range indicate an encoding or transfer error

## Limitations

- PCA-space-based batch effect estimation may fail to capture sophisticated non-linear batch effects, so the baseline can underestimate batch effects on nonlinear structure
- kBET is very sensitive to any kind of bias; a rejection rate of 1 on batch-corrected data requires complementary measures (average silhouette width, PCA-based batch-effect measures)
- kBET has no Python implementation and is available only in R, requiring rpy2 transfer; Linux system is recommended for seamless rpy2 working
- iLISI via evaluate_batch_iLisi is computed only on a randomly chosen test set, not the full data

## Evidence

- [intro] PCA baseline rationale: "Currently, the batch effect estimation techniques mostly depend on the local distribution of cells in the Principal Component (PC) space, which may fail to capture sophisticated non-linear batch"
- [readme] kBET definition: [evidence span withheld — anchor status: external_doc]
- [methods] rpy2 transfer requirement: "To calculate kBET using non-linear embedding, the learned embedding must be transferred to R environment for calculation."
- [methods] iLISI evaluation protocol: "lisi_values = my_model.evaluate_batch_iLisi(Xt,bt,yt,20)"
- [methods] BEENE embedding retrieval: "test_embedding = my_model.get_beene_embeddings(X_test)"
- [other] PCA baseline reproducibility finding: [evidence span withheld — anchor status: not_found]
