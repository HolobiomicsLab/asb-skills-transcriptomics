---
name: kmeans-preclustering-virtual-label-generation
description: 'Use when building the AutoClass composite network: raw scRNA-Seq counts
  (e.g. Dataset1_raw.csv) must be converted into pseudo-labeled training targets for
  the classifier branch because true cell classes are unknown.'
when_to_use_negative:
- Input is already imputed or normalized expression (AutoClassImpute expects the raw
  count matrix and performs its own take_norm step)
- True cell class labels are available and a purely supervised classifier could be
  trained directly, removing the need for virtual labels from pre-clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: AutoClass
  role: Composite autoencoder-plus-classifier network that consumes the K-means virtual
    labels at its bottleneck classifier branch; installed via git clone https://github.com/datapplab/AutoClass
  repo: https://github.com/datapplab/AutoClass
- name: sklearn
  role: Provides the KMeans estimator used for pre-clustering the take_norm-normalized
    matrix into K-1, K, and K+1 clusters
- name: TensorFlow >=2.0
  role: Backend framework in which the autoencoder and classifier branch are built
    and trained with the weighted reconstruction + classification loss
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:92e2b53f021fdb7bafebad002fe0ab01439d4f59632be2e64f70e6019fd7f7aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/datapplab/AutoClass
  tools:
  - AutoClass
  - Python
  - TensorFlow >=2.0
  - numpy
  - pandas
  - sklearn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# K-means pre-clustering virtual label generation

## Summary

Runs K-means on the normalized scRNA-Seq count matrix at K-1, K, and K+1 clusters to generate virtual class labels that supervise the classifier branch attached to the bottleneck layer of the AutoClass autoencoder, blending unsupervised and supervised learning for denoising/imputation.

## When to use

Use when building the AutoClass composite network: raw scRNA-Seq counts (e.g. Dataset1_raw.csv) must be converted into pseudo-labeled training targets for the classifier branch because true cell classes are unknown.

## When NOT to use

- Input is already imputed or normalized expression (AutoClassImpute expects the raw count matrix and performs its own take_norm step)
- True cell class labels are available and a purely supervised classifier could be trained directly, removing the need for virtual labels from pre-clustering

## Inputs

- Raw scRNA-Seq count matrix CSV (e.g. Dataset1_raw.csv, gene-by-cell, index_col=0)
- Log2-normalized matrix produced by take_norm
- Expected number of cell groups K (e.g. six groups in simulated Dataset 1)

## Outputs

- Three sets of virtual class labels from K-means at K-1, K, K+1 clusters
- Averaged imputed expression matrix (res) with the same gene-by-cell dimensions as the input, saved as imputed_matrix.csv

## How to apply

Load the raw count matrix with pandas (index_col=0) and normalize to log2 scale using take_norm. Run sklearn K-means on the normalized data three times with K-1, K, and K+1 clusters to produce three sets of virtual class labels. Build the AutoClass composite network in TensorFlow (>=2.0) with the classifier branch attached to the bottleneck layer, weighting classification loss by the classifier weight w against the reconstruction loss, using dropout_rate=0.3. Run AutoClassImpute once per pre-clustering number and average the three imputed predictions to obtain the final imputed matrix (res), which is saved as imputed_matrix.csv. The rationale, per the paper, is that virtual labels from pre-clustering let the classifier filter noise while retaining signal, and AutoClass is robust to the choice of pre-clustering number.

## Related tools

- **AutoClass** (Composite autoencoder-plus-classifier network that consumes the K-means virtual labels at its bottleneck classifier branch; installed via git clone https://github.com/datapplab/AutoClass) — https://github.com/datapplab/AutoClass
- **sklearn** (Provides the KMeans estimator used for pre-clustering the take_norm-normalized matrix into K-1, K, and K+1 clusters)
- **TensorFlow >=2.0** (Backend framework in which the autoencoder and classifier branch are built and trained with the weighted reconstruction + classification loss)

## Examples

```
expr = pd.read_csv('Dataset1_raw.csv', index_col=0); X = expr.values; X_norm = take_norm(X); labels_km = [KMeans(n_clusters=k, random_state=1).fit_predict(X_norm) for k in (K-1, K, K+1)]
```

## Evaluation signals

- imputed_matrix.csv exists and has the same gene-by-cell dimensions as the raw input matrix
- All imputed values are non-negative
- Downstream fidelity of imputed values can be cross-checked against reference true counts in Dataset1_true.csv
- Robustness check: averaging predictions over K-1, K, K+1 should give stable results given the paper's finding that AutoClass is robust to pre-clustering number
- Training loss should combine reconstruction error and classification loss with classifier weight w, converging with dropout_rate=0.3

## Limitations

- Virtual labels are pseudo-labels from K-means, so clusters only approximate true cell identities; the paper addresses this by averaging over K-1, K, and K+1
- Requires choosing the expected number of groups K, though AutoClass is reported robust to pre-clustering number, bottleneck layer size, and classifier weight
- Requires TensorFlow (>=2.0), numpy, pandas, sklearn, matplotlib installed with Python 3

## Evidence

- [results] AutoClass adds a classifier branch from the bottleneck layer of the autoencoder: [evidence span withheld — anchor status: not_found]
- [abstract] distribution agnostic claim: "AutoClass is robust on key hyperparameter settings including bottleneck layer size, pre-clustering number and classifier weight"
- [methods] installation and dependencies: "AutoClass runs with Python 3, and you need to have TensorFlow (>=2.0) and a few other pacakges installed"
- [results] imputation call with dropout rate: "res = AutoClassImpute(X,dropout_rate=0.3)"
- [methods] repository clone instruction: "git clone https://github.com/datapplab/AutoClass"
