---
name: cytof-dimensionality-reduction-embedding
description: Use when a CyTOF or scRNA-seq expression matrix (e.g. 32-marker BMMC
  data, 265,627 cells) must be projected into a 2D embedding that preserves local
  and global structure and supports out-of-sample transform of held-out cells.
when_to_use_negative:
- When the goal is cell population clustering or label assignment itself rather than
  visualisation/embedding — the Levine benchmark data and PhenoGraph are designed
  for clustering, while ivis produces an embedding
- When the input is already a low-dimensional embedding or PCA score matrix unsuitable
  for KNN-based triplet sampling — ivis expects the scaled high-dimensional feature
  matrix
- When more than 500,000 observations are used without reconsidering the baseline
  model, since the 'maaten' baseline is recommended only for datasets under 500,000
  observations
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ivis
  role: Trains the siamese neural network with a triplet loss to produce the 2D embedding
    and the parametric transform for out-of-sample cells
  repo: https://github.com/beringresearch/ivis
- name: Python
  role: Python interface through which ivis is installed and invoked (also available
    via an R interface)
  repo: https://github.com/beringresearch/ivis
- name: sklearn MinMaxScaler
  role: Scales marker values to the [0, 1] range required before ivis training
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:49403cf557bedd260ded19ec8d270c0c10b349ea745729f574235bf51a185953
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - ivis
  - sklearn MinMaxScaler
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cytof-dimensionality-reduction-embedding

## Summary

Reduce high-dimensional single-cell expression data (CyTOF marker matrices such as Levine_32dim, or PCA-reduced scRNA-seq) to a low-dimensional embedding using ivis, a siamese neural network trained with a triplet loss that preserves both local and global structure. The learned parametric mapping also lets new cells be added to an existing embedding without retraining.

## When to use

Use when a CyTOF or scRNA-seq expression matrix (e.g. 32-marker BMMC data, 265,627 cells) must be projected into a 2D embedding that preserves local and global structure and supports out-of-sample transform of held-out cells.

## When NOT to use

- When the goal is cell population clustering or label assignment itself rather than visualisation/embedding — the Levine benchmark data and PhenoGraph are designed for clustering, while ivis produces an embedding
- When the input is already a low-dimensional embedding or PCA score matrix unsuitable for KNN-based triplet sampling — ivis expects the scaled high-dimensional feature matrix
- When more than 500,000 observations are used without reconsidering the baseline model, since the 'maaten' baseline is recommended only for datasets under 500,000 observations

## Inputs

- CyTOF marker expression matrix (e.g. Levine_32dim: 265,627 cells x 32 surface protein markers from BMMCs, FCS-derived, with manually gated population labels for 104,184 cells)
- scRNA-seq expression matrix (e.g. Melanoma SCP11: 4,645 x 23,686, pre-reduced to n=50 PCs)
- Manually gated cell population labels for supervised or evaluation use
- Held-out cell matrices to be embedded via the parametric transform

## Outputs

- 2D ivis embedding of the training cells (fit_transform output)
- Trained parametric mapping (siamese network) usable via transform() to add new cells to the existing embedding
- Accuracy table of classifier performance on held-out embeddings (e.g. BMMC subsamples of 1,000/5,000/10,000/15,000 cells)
- Distance-matrix correlation value from the Mantel test (Pearson)

## How to apply

Preprocess the marker matrix before embedding: apply the arcsinh transformation X = np.arcsinh(X/5) for Levine CyTOF data, then scale all values to the [0, 1] range with sklearn's MinMaxScaler. Fit ivis on the scaled training data with embedding_dims=2, k=15, and the 'maaten' baseline model, which the paper recommends for datasets with fewer than 500,000 observations; ivis trains a siamese network by sampling positives from each cell's KNN vector and negatives from outside it. To embed held-out cells, pass them through the trained parametric mapping with the model's transform method rather than refitting. Decision points in the loss configuration matter: across pn loss, conventional triplet, and Softmax Ratio losses crossed with Euclidean, Manhattan, and Chebyshev metrics, pn loss with the Euclidean metric gave the best out-of-sample accuracy on BMMC subsamples. Validate either by classification accuracy (train a classifier such as random forest or SVM on the embeddings — the paper used a 75%/25% train/evaluation split) or by a Mantel test of the correlation between original and embedding distance matrices using Pearson correlation.

## Related tools

- **ivis** (Trains the siamese neural network with a triplet loss to produce the 2D embedding and the parametric transform for out-of-sample cells) — https://github.com/beringresearch/ivis
- **Python** (Python interface through which ivis is installed and invoked (also available via an R interface)) — https://github.com/beringresearch/ivis
- **sklearn MinMaxScaler** (Scales marker values to the [0, 1] range required before ivis training)

## Examples

```
from ivis import Ivis; embeddings = Ivis(embedding_dims=2, k=15).fit_transform(X_scaled)
```

## Evaluation signals

- Out-of-sample classification accuracy on held-out embeddings: pn loss with Euclidean metric should reach approximately 0.94, 0.96, 0.97, 0.97 on BMMC subsamples of 1,000, 5,000, 10,000, and 15,000 cells
- Mantel test between the original distance matrix and the embedding-space distance matrices yields a high Pearson correlation, indicating distance preservation
- On synthetic data such as the Smiley dataset, the embedding preserves both the shape and relative locations of each cluster (global structure)
- The transform method adds new data points to the existing embedding without rerunning the fit, and works inside standard sklearn Pipelines
- Verify the embedding is robust to uniform random noise, unlike t-SNE which yields spurious clusters with clean boundaries

## Limitations

- Hyperparameter settings for ivis runs are referenced via Supplementary Figures and the 'maaten' baseline is recommended only for datasets with fewer than 500,000 observations; larger datasets need a different configuration
- In the Levine 32-dim benchmark, only 39% of cells (104,184) carry manually gated population labels, so supervised evaluation is limited to the labelled subset
- Compared algorithms (t-SNE, SIMLR, scvis) behave differently under noise — t-SNE yields spurious clusters with clean boundaries in the presence of random noise, so cross-method comparisons must account for this failure mode

## Evidence

- [abstract] ivis utilizes a siamese neural network architecture that is trained using a novel triplet loss function.: "ivis utilizes a siamese neural network architecture that is trained using a novel triplet loss function."
- [methods] MinMaxScaler scaling before ivis: "Prior to preprocessing, values in both datasets were scaled to [0, 1] range."
- [methods] Accuracy evaluation protocol: "Accuracy was calculated by training a Support Vector Machine classifier on 75% of each subsample and evaluating the classifier performance on the remaining 25%"
- [readme] README description of ivis and its transformer: "Ivis is designed to reduce dimensionality of very large datasets using a siamese neural network trained on triplets."
