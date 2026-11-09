---
name: triplet-loss-implementation
description: 'Use when implementing or reproducing ivis-style triplet-loss dimensionality
  reduction: a siamese network must be built, triplets must be sampled from KNN structure
  (positives from the KNN vector, negatives from outside it), and pn-loss must be
  minimised.'
when_to_use_negative:
- Data has not been scaled to the [0, 1] range and no MinMaxScaler step is planned
  — ivis expects preprocessed values in [0, 1]
- The goal is only to compare off-the-shelf dimensionality-reduction methods (t-SNE,
  LargeVis, SIMLR, scvis) rather than implement ivis's architecture and loss
- Supervised label information is the primary signal and a purely supervised model
  is desired — although ivis supports supervised mode, this skill targets the unsupervised
  triplet-based setup
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: ivis
  role: Framework whose siamese neural network and triplet loss are being implemented;
    supports unsupervised and supervised modes
  repo: https://github.com/beringresearch/ivis
- name: TensorFlow
  role: Backend on which the Keras siamese model and triplet loss run
- name: Annoy
  role: Random projection tree KNN index (AnnoyKnnMatrix-style) used to sample triplet
    positives from the KNN vector and negatives from outside it
- name: Python
  role: Implementation language for the siamese network, loss, and triplet sampler
  repo: https://github.com/beringresearch/ivis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:7ac88b7e77e0886da848aba72f1daa7b09e08d643d2589f5105b79841467e174
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# triplet-loss-implementation

## Summary

Implements ivis's siamese neural network trained with a triplet loss, where approximate nearest-neighbour identification (Annoy random projection trees) supplies the KNN vectors used to sample positive and negative triplets. This is the core training mechanism behind ivis's structure-preserving dimensionality reduction of high-dimensional single-cell datasets.

## When to use

Use when implementing or reproducing ivis-style triplet-loss dimensionality reduction: a siamese network must be built, triplets must be sampled from KNN structure (positives from the KNN vector, negatives from outside it), and pn-loss must be minimised.

## When NOT to use

- Data has not been scaled to the [0, 1] range and no MinMaxScaler step is planned — ivis expects preprocessed values in [0, 1]
- The goal is only to compare off-the-shelf dimensionality-reduction methods (t-SNE, LargeVis, SIMLR, scvis) rather than implement ivis's architecture and loss
- Supervised label information is the primary signal and a purely supervised model is desired — although ivis supports supervised mode, this skill targets the unsupervised triplet-based setup

## Inputs

- Numeric feature matrix (numpy array, sparse matrix, or hdf5 file) scaled to [0, 1] with MinMaxScaler
- Target embedding dimensionality (embedding_dims, e.g. 2)
- Number of nearest neighbours k for triplet sampling (e.g. k=15)
- Annoy KNN index (AnnoyKnnMatrix) built over the input data via random projection trees

## Outputs

- Trained siamese Keras model whose three branches share the 'szubert' base network weights
- Low-dimensional embedding of the input data with the configured embedding_dims columns
- Triplet loss value computed on anchor/positive/negative batches

## How to apply

Build the 'szubert' base network in Keras: three dense layers of 128 neurons with SELU activation and LeCun normal initialisation, followed by a linear-activation embedding layer sized to the target embedding dimensionality (Glorot uniform initialisation). Instantiate three weight-sharing replicates of this base network as the anchor, positive, and negative branches of the siamese architecture. Construct triplets by identifying each point's k nearest neighbours with an AnnoyKnnMatrix-style index (Annoy random projection trees); sample positives from the KNN vector and negatives from outside the KNN vector. Minimise the triplet loss that pulls positives toward the anchor and pushes negatives away, including the positive-negative distance-minimisation term of the pn-loss variant. Scale input data to the [0, 1] range with MinMaxScaler before training, as done in the paper's preprocessing. Validate by compiling the model, running a forward pass on a random batch, and checking that the loss computes without error and the embedding output matches the configured embedding_dims size.

## Related tools

- **ivis** (Framework whose siamese neural network and triplet loss are being implemented; supports unsupervised and supervised modes) — https://github.com/beringresearch/ivis
- **TensorFlow** (Backend on which the Keras siamese model and triplet loss run)
- **Annoy** (Random projection tree KNN index (AnnoyKnnMatrix-style) used to sample triplet positives from the KNN vector and negatives from outside it)
- **Python** (Implementation language for the siamese network, loss, and triplet sampler) — https://github.com/beringresearch/ivis

## Examples

```
from ivis import Ivis; from sklearn.preprocessing import MinMaxScaler; X_scaled = MinMaxScaler().fit_transform(X); model = Ivis(embedding_dims=2, k=15); embeddings = model.fit_transform(X_scaled)
```

## Evaluation signals

- The siamese model compiles and performs a forward pass on a random batch without error
- Embedding output shape equals the configured embedding_dims size
- Positives in each triplet come from the point's KNN vector and negatives from outside the KNN vector, as sampled via the Annoy index
- All three siamese branches share identical weights from the 'szubert' base network (128-neuron SELU dense layers, linear embedding layer)
- Triplet loss (with the pn-loss positive-negative distance-minimisation term) decreases during training on MinMaxScaler-scaled input

## Limitations

- ivis's stochastic components include random initialisation of network weights and approximate (not exact) nearest-neighbour identification using random projection trees, so results may vary between runs
- t-SNE-style alternatives were shown to yield spurious clusters under random noise; verification against such methods requires care since behaviour differs under noise
- Accuracy assessments in the paper relied on subsampling strategies (random stratified subsamples, n=50, of 1000 observations) rather than full-data metrics

## Evidence

- [abstract] ivis utilizes a siamese neural network architecture that is trained using a novel triplet loss function.: "ivis utilizes a siamese neural network architecture that is trained using a novel triplet loss function."
- [discussion] Given that ivis samples positive and negative data-points for each triplet, from the KNN vector and outside the KNN vector respectively: "Given that ivis samples positive and negative data-points for each triplet, from the KNN vector and outside the KNN vector respectively"
- [methods] X = MinMaxScaler().fit_transform(X): "Prior to preprocessing, values in both datasets were scaled to [0, 1] range."
- [readme] Implementation of the ivis algorithm as described in the paper ... using a siamese neural network trained on triplets.: "Ivis is designed to reduce dimensionality of very large datasets using a siamese neural network trained on triplets. Both unsupervised and supervised modes are supported."
- [other] Annoy-based triplet sampling role: [evidence span withheld — anchor status: not_found]
