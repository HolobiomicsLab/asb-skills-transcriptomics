---
name: siamese-network-architecture-construction
description: Use when implementing or reproducing ivis's dimensionality-reduction
  model, i.e. when a scalable neural embedding that preserves local (KNN) and global
  structure must be trained with a triplet loss on high-dimensional single-cell or
  general tabular data.
when_to_use_negative:
- The task requires purely non-parametric embedding without a mapping for new points
  — ivis's value is the parametric transform; if no out-of-sample embedding is needed
  and only pairwise similarity preservation matters, alternatives may suffice
- Labels are available and supervised classification is the goal — although ivis supports
  supervised mode, this skill covers the unsupervised triplet-based siamese construction
  only
- The dataset is tiny and low-dimensional where simpler methods (e.g. on the iris-scale
  toy data) do not need a neural siamese network and Annoy ANN indexing overhead
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: ivis
  role: Provides the siamese neural network implementation trained with triplet loss
    for dimensionality reduction
  repo: https://github.com/beringresearch/ivis
- name: Python
  role: Language interface through which ivis and its TensorFlow/Keras siamese network
    are used
- name: TensorFlow
  role: Backend on which the ivis siamese network and triplet loss run
- name: Annoy
  role: AnnoyKnnMatrix-style random projection tree index supplying approximate nearest
    neighbours for positive/negative triplet sampling
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:12f3b426aa9a26e6eb99c97bb0a2b619b53426f5fb796a59521719f10ccd3dfd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Annoy
  - Python
  - TensorFlow
  - ivis
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# siamese-network-architecture-construction

## Summary

Constructs the ivis siamese neural network: three identical Keras replicates of a 'szubert' base network (dense 128-neuron SELU layers with LeCun normal init, plus a linear-activation embedding layer of embedding_dims size) with shared weights, trained on triplets sampled via approximate nearest-neighbour identification (Annoy random projection trees).

## When to use

Use when implementing or reproducing ivis's dimensionality-reduction model, i.e. when a scalable neural embedding that preserves local (KNN) and global structure must be trained with a triplet loss on high-dimensional single-cell or general tabular data.

## When NOT to use

- The task requires purely non-parametric embedding without a mapping for new points — ivis's value is the parametric transform; if no out-of-sample embedding is needed and only pairwise similarity preservation matters, alternatives may suffice
- Labels are available and supervised classification is the goal — although ivis supports supervised mode, this skill covers the unsupervised triplet-based siamese construction only
- The dataset is tiny and low-dimensional where simpler methods (e.g. on the iris-scale toy data) do not need a neural siamese network and Annoy ANN indexing overhead

## Inputs

- Feature matrix X (numpy array, sparse matrix, or hdf5 file) of shape n_observations x n_features
- Data scaled to [0, 1] range (e.g. via sklearn MinMaxScaler)
- Hyperparameters: embedding_dims (target embedding dimensionality), k (number of nearest neighbours for triplet sampling, e.g. 15)

## Outputs

- Compiled siamese Keras/TensorFlow model with three weight-shared base-network replicates
- Low-dimensional embedding of shape n_observations x embedding_dims produced by fit_transform
- Parametric mapping function supporting transform() to embed new data points without rerunning the algorithm

## How to apply

Build the 'szubert' base network in Keras/TensorFlow: three dense layers of 128 neurons each using SELU activation with LeCun normal weight initialisation, followed by a linear-activation embedding layer (Glorot uniform init) whose output size equals the configured embedding_dims (e.g. 2). Instantiate three identical replicates sharing weights so anchor, positive, and negative inputs pass through the same function, and assemble them into the siamese architecture. Train with a triplet loss that pulls positives toward the anchor and pushes negatives away, including the positive-negative distance-minimisation term of the pn-loss variant. Supply triplets by sampling positives from each point's approximate k-nearest-neighbour vector and negatives from outside it, using an AnnoyKnnMatrix-style index built with Annoy random projection trees (e.g. k=15, as in the README example). Scale inputs to [0, 1] (e.g. MinMaxScaler) before fitting, as the paper's preprocessing scales values to the [0, 1] range. Validate by compiling the model, running a forward pass on a random batch, and confirming the loss computes and the embedding output has the configured embedding_dims dimensionality.

## Related tools

- **ivis** (Provides the siamese neural network implementation trained with triplet loss for dimensionality reduction) — https://github.com/beringresearch/ivis
- **Python** (Language interface through which ivis and its TensorFlow/Keras siamese network are used)
- **TensorFlow** (Backend on which the ivis siamese network and triplet loss run)
- **Annoy** (AnnoyKnnMatrix-style random projection tree index supplying approximate nearest neighbours for positive/negative triplet sampling)

## Examples

```
from ivis import Ivis; from sklearn.preprocessing import MinMaxScaler; X_scaled = MinMaxScaler().fit_transform(X); embeddings = Ivis(embedding_dims=2, k=15).fit_transform(X_scaled)
```

## Evaluation signals

- Model compiles without error and a forward pass on a random batch returns a loss value
- Embedding output has exactly the configured embedding_dims dimensionality (e.g. 2)
- Each of the three base-network replicates shares identical weights (anchor, positive, negative inputs use the same function)
- Triplet positives are drawn from each point's KNN vector and negatives from outside it, consistent with the Annoy-based k-NN index (k=15 in the reference example)
- Downstream validation: ivis embedding preserves global structure (e.g. Mantel test/Pearson correlation between original and embedded distance matrices)

## Limitations

- Stochastic components include approximate k-NN identification via random projection trees and random initialisation of network weights, so results vary between runs
- Neighbour identification is approximate (Annoy random projection trees), so triplet sampling may miss true nearest neighbours
- Training depends on hyperparameter choices (embedding_dims, k); the article notes robustness to hyperparameter settings was assessed only via supplementary figures rather than explicit values in the main text

## Evidence

- [abstract] ivis operates as a siamese neural network trained with a triplet loss: "ivis utilizes a siamese neural network architecture that is trained using a novel triplet loss function."
- [discussion] ivis samples positive and negative data-points for each triplet, from the KNN vector and outside the KNN vector respectively: "Given that ivis samples positive and negative data-points for each triplet, from the KNN vector and outside the KNN vector respectively"
- [methods] Values scaled to [0, 1] range before preprocessing: "Prior to preprocessing, values in both datasets were scaled to [0, 1] range."
- [readme] ivis README: siamese neural network trained on triplets, runs on TensorFlow: "Ivis is designed to reduce dimensionality of very large datasets using a siamese neural network trained on triplets."
- [readme] README example invocation with MinMaxScaler, embedding_dims=2, k=15: "X_scaled = MinMaxScaler().fit_transform(X)

model = Ivis(embedding_dims=2, k=15)"
