---
name: annoy-knn-triplet-sampling
description: Use when implementing or reproducing ivis-style siamese triplet training
  and you need positives drawn from each point's KNN vector and negatives from outside
  it, via approximate KNN search (AnnoyKnnMatrix).
when_to_use_negative:
- The task requires exact (not approximate) nearest-neighbour retrieval for downstream
  distance computations — Annoy random projection trees are approximate and stochastic
- Input data is not yet scaled — ivis preprocessing requires values scaled to the
  [0, 1] range before KNN/triplet sampling
- Supervised labels are available and a supervised ivis mode is intended, where triplet
  sampling is driven by labels rather than unsupervised KNN structure
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0091
tools:
- name: ivis
  role: Siamese neural network framework whose triplet sampling consumes the Annoy
    KNN index; trained with a novel triplet loss
  repo: https://github.com/beringresearch/ivis
- name: Annoy
  role: Random projection tree index used for approximate identification of k-nearest
    neighbours that supply the neighbourhood information for triplet sampling
- name: TensorFlow
  role: Backend on which the ivis siamese network and triplet loss are implemented
    and run
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:7907ae63d6a9899cc1dc0aa4f215ffe0e3c2db9f701c43ca88840861d219e3f9
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
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# annoy-knn-triplet-sampling

## Summary

Approximate nearest-neighbour identification (Annoy random projection trees) supplies the neighbourhood structure used to sample anchor-positive-negative triplets for training ivis's siamese neural network with a triplet/pn loss. It is the stochastic neighbourhood step that makes scalable triplet-based dimensionality reduction tractable.

## When to use

Use when implementing or reproducing ivis-style siamese triplet training and you need positives drawn from each point's KNN vector and negatives from outside it, via approximate KNN search (AnnoyKnnMatrix).

## When NOT to use

- The task requires exact (not approximate) nearest-neighbour retrieval for downstream distance computations — Annoy random projection trees are approximate and stochastic
- Input data is not yet scaled — ivis preprocessing requires values scaled to the [0, 1] range before KNN/triplet sampling
- Supervised labels are available and a supervised ivis mode is intended, where triplet sampling is driven by labels rather than unsupervised KNN structure

## Inputs

- Feature matrix (numpy array, sparse matrix, or hdf5 file) scaled to [0, 1] with MinMaxScaler
- Number of nearest neighbours k (e.g. k=15)
- Target embedding dimensionality (embedding_dims, e.g. 2)

## Outputs

- AnnoyKnnMatrix-style approximate KNN index over the input observations
- Sampled (anchor, positive, negative) triplets: positives from each point's KNN vector, negatives from outside it
- Trained siamese network producing embeddings of size embedding_dims

## How to apply

Build an AnnoyKnnMatrix-style index over the (MinMaxScaler-scaled, typically [0,1]) input matrix using Annoy random projection trees to retrieve the k-nearest neighbours of every observation (k=15 in the ivis README example). For each anchor point, draw positives from its KNN vector and negatives from outside the KNN vector, then feed the (anchor, positive, negative) triplets into the siamese network whose three weight-shared branches are the 'szubert' base: three 128-neuron SELU dense layers (LeCun normal init) followed by a linear-activation embedding layer with Glorot uniform init sized to embedding_dims. Train with the triplet loss that pulls positives toward the anchor and pushes negatives away, including the positive-negative distance-minimization term of the pn-loss variant. The approximate (random projection tree) KNN search is what keeps sampling stochastic and scales to hundreds of thousands of cells or millions of observations.

## Related tools

- **ivis** (Siamese neural network framework whose triplet sampling consumes the Annoy KNN index; trained with a novel triplet loss) — https://github.com/beringresearch/ivis
- **Annoy** (Random projection tree index used for approximate identification of k-nearest neighbours that supply the neighbourhood information for triplet sampling)
- **TensorFlow** (Backend on which the ivis siamese network and triplet loss are implemented and run)

## Examples

```
from ivis import Ivis; from sklearn.preprocessing import MinMaxScaler; X_scaled = MinMaxScaler().fit_transform(X); embeddings = Ivis(embedding_dims=2, k=15).fit_transform(X_scaled)
```

## Evaluation signals

- The siamese model compiles and a forward pass on a random batch computes the triplet loss without error
- The embedding output shape equals the configured embedding_dims (e.g. 2)
- For sampled triplets, positives originate from each anchor's KNN vector and negatives from outside it
- Training reduces the triplet loss (positives pulled toward anchors, negatives pushed away, pn-loss term minimized)
- Resulting embeddings preserve local and global structure, e.g. verified by Mantel test correlation between original and embedded distance matrices

## Limitations

- KNN identification is approximate (random projection trees), so sampled neighbourhoods — and hence triplets — are stochastic rather than exact
- Neural network weight initialisation is random, adding further stochasticity to the training outcome
- ivis is robust to hyperparameter settings, but hyperparameter details are referenced only via supplementary figures in the discussion

## Evidence

- [discussion] Triplet sampling from KNN vector (positive) and outside KNN vector (negative): "Given that ivis samples positive and negative data-points for each triplet, from the KNN vector and outside the KNN vector respectively"
- [abstract] ivis uses a siamese network trained with a triplet loss: "ivis utilizes a siamese neural network architecture that is trained using a novel triplet loss function."
- [readme] ivis runs on top of TensorFlow with Ivis constructor: [evidence span withheld — anchor status: external_doc]
- [readme] Ivis constructor with embedding_dims and k parameters: "model = Ivis(embedding_dims=2, k=15)"
- [methods] Scaling precondition for ivis inputs: "X = MinMaxScaler().fit_transform(X)"
