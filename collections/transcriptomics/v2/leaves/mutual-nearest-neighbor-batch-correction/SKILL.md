---
name: mutual-nearest-neighbor-batch-correction
description: Use when extending cluster labels from an FFC-trained dataset to newly
  arriving (out-of-sample or streaming) test batches that show batch effects relative
  to the training data, prior to computing the joint affinity matrix.
when_to_use_negative:
- Training and testing data are already in the same batch condition — MNN correction
  is only needed to correct batch effects between training and testing data.
- Only a single in-sample cluster exists in the data — the article notes every data
  point can only receive one label in that case, and Monte Carlo validation cannot
  estimate uncertainty.
- High-dimensional raw input without nonlinear dimensionality reduction — the method
  should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE,
  PHATE) because it mimics a low-dimensional fire propagation process.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: forest-fire-clustering (FFC)
  role: Provides the training embeddings and inductive online-learning framework into
    which the batch-corrected test data are merged for label extension
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: scanpy
  role: Python package used for efficient neighbor-based operations supporting the
    affinity construction in the online inference workflow
- name: scikit-learn (sklearn)
  role: Generates the synthetic Gaussian training/test clusters used in the online
    out-of-sample cluster discovery benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:0f52fcb02b5d239c4905713040d810f15acfb6854dd88d44760c5224743a1096
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Python
  - forest-fire-clustering (FFC)
  - numba
  - numpy
  - scanpy
  - scikit-learn (sklearn)
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# mutual-nearest-neighbor-batch-correction

## Summary

Applies mutual nearest neighbor (MNN) batch correction between a Forest Fire Clustering training dataset and incoming out-of-sample testing data, so that the joint affinity matrix used for online label propagation is computed on comparably aligned feature spaces.

## When to use

Use when extending cluster labels from an FFC-trained dataset to newly arriving (out-of-sample or streaming) test batches that show batch effects relative to the training data, prior to computing the joint affinity matrix.

## When NOT to use

- Training and testing data are already in the same batch condition — MNN correction is only needed to correct batch effects between training and testing data.
- Only a single in-sample cluster exists in the data — the article notes every data point can only receive one label in that case, and Monte Carlo validation cannot estimate uncertainty.
- High-dimensional raw input without nonlinear dimensionality reduction — the method should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE) because it mimics a low-dimensional fire propagation process.

## Inputs

- Low-dimensional training embeddings from FFC preprocessing (sample x feature matrix)
- Out-of-sample testing data batches projected onto the training embeddings
- In-sample cluster labels from cluster_obj.cluster_labels

## Outputs

- Batch-corrected joint representation of training and testing points
- Joint affinity matrix of training and testing data
- Extended (out-of-sample) cluster labels for the testing points

## How to apply

After fitting FFC on training data (cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100)), project each testing batch onto the low-dimensional representations (embeddings) learned from the training dataset. Because training and testing data may exhibit batch effects, apply mutual nearest neighbor batch correction between the training and testing data, matching pairs of cells that are mutual nearest neighbors across the two datasets to align the batches. Then compute the joint affinity matrix of the training and testing points on the corrected joint representation, and use it to extend (propagate) the in-sample cluster labels to the out-of-sample points. Score the extended labels against reference labels with adjusted Rand index (ARI) and purity; in the paper's online-learning benchmark (four training Gaussian clusters, sigma = 0.1, streaming batches with increasing unseen clusters and Gaussian noise), robust performance corresponds to average ARI and purity both above 0.90.

## Related tools

- **forest-fire-clustering (FFC)** (Provides the training embeddings and inductive online-learning framework into which the batch-corrected test data are merged for label extension) — https://github.com/gersteinlab/forest-fire-clustering
- **scanpy** (Python package used for efficient neighbor-based operations supporting the affinity construction in the online inference workflow)
- **scikit-learn (sklearn)** (Generates the synthetic Gaussian training/test clusters used in the online out-of-sample cluster discovery benchmark)

## Evaluation signals

- Extended out-of-sample labels match the training cluster labels' structure: reproduced average ARI and purity both exceed 0.90 as the number of unseen clusters and Gaussian noise increase
- Test batches are verified to lie in the same low-dimensional space as training data (projected onto training representations) before the joint affinity matrix is computed
- After MNN correction, mutual nearest neighbor pairs span training and testing batches, indicating batch effects were corrected
- Joint affinity matrix has nonzero affinities connecting training and testing points, enabling label propagation to every test point

## Limitations

- Curse of dimensionality: clustering and inference degrade as distances become more uniform in high dimensions
- The method is sensitive to high dimensionality because it mimics a low-dimensional fire propagation process and must be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)
- Distance matrices for datasets over 1 million points are computationally expensive and must be approximated via KNN

## Evidence

- [discussion] MNN correction precedes joint affinity computation: "we correct batch effects between the training and testing data with mutual nearest neighbor batch correction"
- [discussion] Projection and joint affinity matrix in online clustering: "the testing data are projected onto the low-dimensional representations of the training dataset. A joint affinity matrix of the training and testing data is then calculated"
- [readme] Inductive online inference motivation: "our inductive algorithm is able to make robust inferences in an online-learning context."
