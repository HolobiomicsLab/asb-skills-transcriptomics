---
name: keras-model-assembly
description: Use when implementing ivis-style or other triplet-loss siamese models
  in TensorFlow/Keras and you need weight-shared anchor/positive/negative branches
  built from a shared base encoder.
when_to_use_negative:
- Task is unsupervised embedding with a non-triplet objective (e.g. plain autoencoder
  or PCA reduction such as the article's PCA to n=50 for Melanoma features) — weight-shared
  triplet branches are unnecessary.
- Data is not scaled suitably for SELU-based dense networks; ivis workflows scale
  inputs to [0,1] with MinMaxScaler first, so raw unnormalised features may train
  poorly.
- Only a single-input encoder is needed (no anchor/positive/negative triplet structure),
  e.g. a supervised classifier head.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: ivis
  role: Reference implementation whose siamese triplet-loss architecture this assembly
    reproduces; provides the 'szubert' base network pattern
  repo: https://github.com/beringresearch/ivis
- name: TensorFlow
  role: Backend on which ivis runs; used to build and compile the Keras siamese model
- name: Python
  role: Implementation language for the Keras model assembly and forward-pass validation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:5e07dad868866a5a48f2ff342ffe6dd55899f9b311b5817adc1ac132d3673f44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - TensorFlow
  - ivis
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# keras-model-assembly

## Summary

Assembles a siamese neural network in Keras by instantiating three weight-shared replicas of a dense base network ('szubert': 3×128-neuron SELU layers with LeCun normal init, plus a linear-activation embedding layer with Glorot uniform init sized to embedding_dims), so anchor, positive, and negative inputs pass through identical weights for triplet-loss training as used in ivis.

## When to use

Use when implementing ivis-style or other triplet-loss siamese models in TensorFlow/Keras and you need weight-shared anchor/positive/negative branches built from a shared base encoder.

## When NOT to use

- Task is unsupervised embedding with a non-triplet objective (e.g. plain autoencoder or PCA reduction such as the article's PCA to n=50 for Melanoma features) — weight-shared triplet branches are unnecessary.
- Data is not scaled suitably for SELU-based dense networks; ivis workflows scale inputs to [0,1] with MinMaxScaler first, so raw unnormalised features may train poorly.
- Only a single-input encoder is needed (no anchor/positive/negative triplet structure), e.g. a supervised classifier head.

## Inputs

- Input tensors for anchor, positive, and negative samples (e.g. MinMaxScaler-scaled [0,1] feature arrays)
- Target embedding dimensionality (embedding_dims)
- KNN index or neighbour matrix (e.g. AnnoyKnnMatrix) supplying triplet positives/negatives

## Outputs

- Compiled siamese Keras model with three weight-shared branches
- Low-dimensional embedding vectors of size embedding_dims per input

## How to apply

Build the base network in Keras: three dense layers of 128 neurons each using SELU activation with LeCun normal weight initialization, followed by a linear-activation embedding layer with Glorot uniform initialization whose output size equals the target embedding dimensionality (e.g. embedding_dims=2). Instantiate three identical copies of this base network and feed the anchor, positive, and negative inputs through them so all three branches share the same weights — this weight sharing is what makes the architecture siamese. Wire the three branch outputs into a triplet loss that pulls positive neighbours toward the anchor, pushes negatives away, and includes the positive-negative distance-minimization term of the pn-loss variant. Compile with TensorFlow and validate by performing a forward pass on a random batch, confirming the loss computes and the embedding output has the configured embedding_dims size. Positives for training are supplied by approximate nearest-neighbour sampling (e.g. AnnoyKnnMatrix random projection trees), not by the assembly step itself.

## Related tools

- **ivis** (Reference implementation whose siamese triplet-loss architecture this assembly reproduces; provides the 'szubert' base network pattern) — https://github.com/beringresearch/ivis
- **TensorFlow** (Backend on which ivis runs; used to build and compile the Keras siamese model)
- **Python** (Implementation language for the Keras model assembly and forward-pass validation)

## Examples

```
from ivis import Ivis; from sklearn.preprocessing import MinMaxScaler; X_scaled = MinMaxScaler().fit_transform(X); model = Ivis(embedding_dims=2, k=15); embeddings = model.fit_transform(X_scaled)
```

## Evaluation signals

- Forward pass on a random batch computes the triplet loss without error
- Embedding output tensor has the configured embedding_dims size (e.g. 2)
- All three branches (anchor/positive/negative) reference the same base-network weights, i.e. weight sharing is verified in the model graph
- Training with KNN-sampled triplets (positives from each point's KNN vector, negatives from outside it) reduces the triplet/pn loss over epochs

## Limitations

- Random initialisation of network weights is a stochastic component, so results vary between runs unless seeds are fixed
- Nearest-neighbour triplet sampling depends on approximate KNN identification (random projection trees / Annoy); poor neighbourhood estimates degrade triplet quality
- Hyperparameter settings for ivis runs may only be referenced via supplementary figures rather than stated explicitly in the text

## Evidence

- [other] Build the 'szubert' base network in Keras: three dense layers of 128 neurons each using SELU activation with LeCun normal weight initialization: [evidence span withheld — anchor status: not_found]
- [discussion] Triplet sampling from KNN vector (positive) and outside KNN vector (negative): "ivis samples positive and negative data-points for each triplet, from the KNN vector and outside the KNN vector respectively"
- [abstract] Siamese architecture trained with triplet loss: "ivis utilizes a siamese neural network architecture that is trained using a novel triplet loss function."
- [readme] Ivis runs on top of TensorFlow with siamese network trained on triplets: "Ivis is designed to reduce dimensionality of very large datasets using a siamese neural network trained on triplets."
- [readme] Ivis installation on TensorFlow: [evidence span withheld — anchor status: external_doc]
