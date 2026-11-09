---
name: triplet-loss-siamese-network-training
description: Use when you need to embed high-dimensional single-cell expression data
  (e.g. CyTOF BMMC marker panels or scRNA-seq) into 2D while preserving local and
  global structure, and when new cells must be added to an existing embedding without
  rerunning the algorithm.
when_to_use_negative:
- When only local cluster separation matters and a well-established t-SNE workflow
  is already validated — though note t-SNE yields spurious clusters under uniform
  random noise
- When the input is a small low-dimensional dataset where simpler embeddings suffice
  (ivis still performs well on toy data like iris, but adds deep-learning overhead)
- When no feature matrix is available (e.g. raw FCS files must first be converted/transformed,
  e.g. arcsinh-transformed, before ivis can be applied)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3306
tools:
- name: ivis
  role: Python/R implementation of the siamese neural network trained on triplets
    used to compute the low-dimensional embedding and out-of-sample transform
  repo: https://github.com/beringresearch/ivis
- name: Python
  role: Runtime environment and interface for installing ivis (pip install ivis[cpu])
    and running the embedding workflow
  repo: https://github.com/beringresearch/ivis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:208e661f119f41c03e8d875e5c8b795ff5db8ab41760a7f8c1f70f4b72bcfb3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - ivis
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3306
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# triplet-loss-siamese-network-training

## Summary

Train a siamese neural network with a triplet loss function to reduce dimensionality of high-dimensional single-cell datasets, using ivis's novel triplet loss that samples positives from the KNN vector and negatives from outside it. This is the core mechanism by which ivis captures both local and global structure while learning a parametric mapping for out-of-sample embedding.

## When to use

Use when you need to embed high-dimensional single-cell expression data (e.g. CyTOF BMMC marker panels or scRNA-seq) into 2D while preserving local and global structure, and when new cells must be added to an existing embedding without rerunning the algorithm.

## When NOT to use

- When only local cluster separation matters and a well-established t-SNE workflow is already validated — though note t-SNE yields spurious clusters under uniform random noise
- When the input is a small low-dimensional dataset where simpler embeddings suffice (ivis still performs well on toy data like iris, but adds deep-learning overhead)
- When no feature matrix is available (e.g. raw FCS files must first be converted/transformed, e.g. arcsinh-transformed, before ivis can be applied)

## Inputs

- High-dimensional feature matrix (numpy array, sparse matrix, or hdf5 file) of single-cell expression values, e.g. BMMC CyTOF 32-marker data from Levine_32dim
- Scalar labels or cell-type annotations for supervised mode
- kNN graph parameter k (e.g. k=15) and embedding_dims (e.g. 2)

## Outputs

- Low-dimensional (e.g. 2D) embedding of the training observations
- Trained parametric mapping supporting out-of-sample embedding of new cells via a transform method
- Optionally, loss/distance-metric accuracy tables (e.g. 8x4 CSV of classification accuracies across subsample sizes)

## How to apply

Preprocess the expression matrix with MinMaxScaler so values are in the [0, 1] range (for CyTOF data such as Levine-32-dim, apply an arcsinh transform X = np.arcsinh(X/5) before scaling). Instantiate ivis with k=15 nearest neighbours and the 'maaten' baseline model (recommended defaults for datasets with <500,000 observations) and fit on the scaled data; the siamese network samples positive neighbours from each cell's KNN vector and negatives from outside it and minimises the triplet loss. Optionally choose between pn loss, conventional triplet loss, and Softmax Ratio loss, each crossed with Euclidean, Manhattan, or Chebyshev distance metrics in the triplet-loss objective. Because the trained network is a parametric mapping, transform held-out cells via model.transform to obtain out-of-sample embeddings without refitting. Validate by assessing preserved global structure (e.g. Mantel test with Pearson correlation of distance matrices) or by training a classifier on embeddings and measuring held-out accuracy.

## Related tools

- **ivis** (Python/R implementation of the siamese neural network trained on triplets used to compute the low-dimensional embedding and out-of-sample transform) — https://github.com/beringresearch/ivis
- **Python** (Runtime environment and interface for installing ivis (pip install ivis[cpu]) and running the embedding workflow) — https://github.com/beringresearch/ivis

## Examples

```
from ivis import Ivis; from sklearn.preprocessing import MinMaxScaler; X = MinMaxScaler().fit_transform(X); model = Ivis(embedding_dims=2, k=15); embeddings = model.fit_transform(X); new_emb = model.transform(X_new)
```

## Evaluation signals

- Mantel test Pearson correlation between original and embedding distance matrices confirms preserved global structure
- Out-of-sample classification accuracy pattern reproduced: Euclidean pn loss on BMMC subsamples yields ~0.94/0.96/0.97/0.97 at 1,000/5,000/10,000/15,000 cells
- Embedding retains both local cluster separation and global relative cluster locations (e.g. cluster shapes preserved on the synthetic Smiley dataset)
- Model.transform produces valid embeddings for held-out cells without refitting, compatible with sklearn Pipelines
- Input matrix is verified to be scaled to the [0, 1] range before fit

## Limitations

- t-SNE-style algorithms (and pairwise-(dis)similarity methods like LargeVis) can produce spurious clusters with clean boundaries in the presence of random noise; ivis is reported robust here but results should be inspected
- Accuracy of embeddings depends on hyperparameter choices (k, model/baseline, loss, metric); the article references hyperparameter sensitivity via Supplementary Figures rather than exhaustive settings in the main text
- Recommended defaults (k=15, 'maaten' model) apply to datasets with <500,000 observations; very large datasets may need different settings
- Raw CyTOF data requires arcsinh transformation (X/5) and MinMaxScaler scaling to [0,1] before training; skipping preprocessing harms embedding quality

## Evidence

- [abstract] siamese neural network trained on triplets: "ivis utilizes a siamese neural network architecture that is trained using a novel triplet loss function."
- [discussion] ivis samples positive and negative data-points for each triplet, from the KNN vector and outside the KNN vector respectively: "Given that ivis samples positive and negative data-points for each triplet, from the KNN vector and outside the KNN vector respectively"
- [methods] values in both datasets were scaled to [0, 1] range: "Prior to preprocessing, values in both datasets were scaled to [0, 1] range."
- [readme] Ivis runs on top of TensorFlow: [evidence span withheld — anchor status: external_doc]
- [intro] parametric mapping allows adding new data points without rerunning: "ivis learns a parametric mapping from the high-dimensional space to low-dimensional embedding, facilitating seamless addition of new data points to the mapping function."
