---
name: neural-network-embedding-training
description: Use when a high-dimensional single-cell (scRNA-seq or CyTOF) expression
  matrix must be reduced to a low-dimensional embedding at scale — especially >100k
  cells where t-SNE is too slow.
when_to_use_negative:
- The goal is strict local-structure visualization of a small dataset (<100k cells)
  where standard Barnes-Hut t-SNE completes in reasonable time
- The input is a low-dimensional toy dataset with few features where the siamese network's
  overhead offers no benefit (though ivis still works, e.g. iris)
- 'Random-noise robustness patterns matter: t-SNE yields spurious clean-boundary clusters
  under noise, so comparative interpretation of cluster boundaries requires care'
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: ivis
  role: Siamese neural network trained with triplet loss that performs the dimensionality
    reduction; installed via pip install ivis[cpu] or ivis[gpu], runs on TensorFlow
  repo: https://github.com/beringresearch/ivis
- name: scikit-learn
  role: Provides MinMaxScaler for the [0,1] preprocessing step and the Barnes-Hut
    t-SNE baseline used for runtime comparison
- name: t-SNE
  role: Baseline embedding method compared against ivis for runtime and global-structure
    preservation
- name: Python
  role: Language interface in which ivis is implemented and invoked
  repo: https://github.com/beringresearch/ivis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:811fd73fcc00969370c3ed09f3103b53873b79bf02c0920a91ed379370616803
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - ivis
  - scikit-learn
  - t-SNE
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# neural-network-embedding-training

## Summary

Trains a siamese neural network with a triplet loss function (via the ivis algorithm) to produce low-dimensional embeddings of high-dimensional single-cell expression data, preserving both local and global structure.

## When to use

Use when a high-dimensional single-cell (scRNA-seq or CyTOF) expression matrix must be reduced to a low-dimensional embedding at scale — especially >100k cells where t-SNE is too slow.

## When NOT to use

- The goal is strict local-structure visualization of a small dataset (<100k cells) where standard Barnes-Hut t-SNE completes in reasonable time
- The input is a low-dimensional toy dataset with few features where the siamese network's overhead offers no benefit (though ivis still works, e.g. iris)
- Random-noise robustness patterns matter: t-SNE yields spurious clean-boundary clusters under noise, so comparative interpretation of cluster boundaries requires care

## Inputs

- High-dimensional expression matrix (scRNA-seq counts, CyTOF protein intensities) as numpy array, sparse matrix, or HDF5 file
- Optional: manually gated cluster labels (e.g. Levine-32-dim, 32 protein markers, 265,627 cells)
- Hyperparameters: embedding_dims, k (KNN size, e.g. k=15), model, n_epochs_without_progress

## Outputs

- Low-dimensional embedding matrix (embedding_dims=2 by default) with one coordinate pair per observation
- Trained siamese network supporting transform() to add new data points to the existing embedding
- Training runtime metrics for scaling benchmarks (e.g. written to runtime_benchmark_results.csv)

## How to apply

Scale the expression matrix to the [0, 1] range with sklearn's MinMaxScaler (optionally arcsinh-transform CyTOF data as np.arcsinh(X/5) beforehand), then fit an ivis siamese neural network using triplet loss, sampling positives from the KNN vector and negatives from outside it. For large datasets (>1M observations), use the recommended hyperparameters k=15-150, n_epochs_without_progress=2-3, model='szubert'. Record training time to verify linear scaling — ivis produced embeddings of the 1.3M-cell 10X mouse brain dataset in under 30 minutes while scikit-learn Barnes-Hut t-SNE exceeded 24 hours. Validate embedding quality by checking that global structure (relative cluster positions) is preserved, e.g. via a Mantel test with Pearson correlation on distance matrices.

## Related tools

- **ivis** (Siamese neural network trained with triplet loss that performs the dimensionality reduction; installed via pip install ivis[cpu] or ivis[gpu], runs on TensorFlow) — https://github.com/beringresearch/ivis
- **scikit-learn** (Provides MinMaxScaler for the [0,1] preprocessing step and the Barnes-Hut t-SNE baseline used for runtime comparison)
- **t-SNE** (Baseline embedding method compared against ivis for runtime and global-structure preservation)
- **Python** (Language interface in which ivis is implemented and invoked) — https://github.com/beringresearch/ivis

## Examples

```
X_scaled = MinMaxScaler().fit_transform(X); model = Ivis(embedding_dims=2, k=15); embeddings = model.fit_transform(X_scaled)
```

## Evaluation signals

- Training completes on the full dataset without subsampling; on the 1.3M-cell 10X mouse brain data, ivis runtime should be under 30 minutes (t-SNE baseline exceeds 24 hours)
- Embedding has 2 dimensions (embedding_dims=2) and one row per input observation
- Global structure check: relative locations/shapes of clusters are preserved in the embedding (e.g. Smiley dataset behavior)
- Mantel test with Pearson correlation between original and embedding distance matrices yields a positive correlation
- model.transform(X_new) maps previously unseen points into the existing embedding without retraining

## Limitations

- scikit-learn Barnes-Hut t-SNE did not finish within 24 hours on 1.3M cells, so head-to-head runtime comparisons at that scale require a timeout policy
- ivis results are robust but hyperparameter settings (k, n_epochs_without_progress, model) should follow the supplementary hyperparameter table binned by observation count
- t-SNE's spurious clusters with clean boundaries under random noise mean visual cluster separation alone is not a reliable quality signal for any embedding method

## Evidence

- [abstract] ivis utilizes a siamese neural network architecture that is trained using a novel triplet loss function.: "ivis utilizes a siamese neural network architecture that is trained using a novel triplet loss function."
- [methods] MinMax scaling preprocessing: "X = MinMaxScaler().fit_transform(X)"
- [discussion] ivis samples positives from KNN, negatives from outside: "Given that ivis samples positive and negative data-points for each triplet, from the KNN vector and outside the KNN vector respectively"
- [readme] installation via pip on TensorFlow: [evidence span withheld — anchor status: external_doc]
- [readme] scalable to millions of observations: [evidence span withheld — anchor status: external_doc]
