---
name: out-of-sample-embedding-prediction-via-parametric-mapping
description: Use when labeled/unlabeled cells arrive after a dimensionality-reduction
  model was fit and you need to map them into the existing embedding (no refitting)
  and quantify how well the mapping generalizes as a function of training-set size.
when_to_use_negative:
- When the goal is a one-off visualization of a fixed dataset with no new cells to
  project — plain fit_transform is sufficient and the parametric mapping is not being
  exercised
- When labels are available for the new cells and you could retrain ivis on the full
  data — re-embedding avoids any extrapolation error from the parametric mapping
- 'When features are unscaled/raw: ivis assumes inputs preprocessed to the [0, 1]
  range (e.g. with MinMaxScaler, or arcsinh-transformed for CyTOF data), and out-of-range
  features invalidate the mapping'
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: ivis
  role: Siamese neural network trained with triplet loss (KNN-vector positives, outside-KNN-vector
    negatives, k=15) whose parametric mapping projects out-of-sample cells into the
    existing embedding via transform
  repo: https://github.com/beringresearch/ivis
- name: Python
  role: Implementation environment for MinMaxScaler preprocessing, ivis training/transform,
    and random forest classification
  repo: https://github.com/beringresearch/ivis
- name: R
  role: Alternative interface for running ivis and the transform-based out-of-sample
    projection
  repo: https://github.com/beringresearch/ivis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:817a23163d6a41c7317300ec6f31199baedc983c471af411edbd26405906ce7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - R
  - ivis
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# out-of-sample-embedding-prediction-via-parametric-mapping

## Summary

Projects new (out-of-sample) cells into an existing ivis low-dimensional embedding using ivis's parametric neural-network mapping (the `transform` method), without retraining, then evaluates the embedding by classifying projected cells (e.g. random forest on BMMC cell-type labels).

## When to use

Use when labeled/unlabeled cells arrive after a dimensionality-reduction model was fit and you need to map them into the existing embedding (no refitting) and quantify how well the mapping generalizes as a function of training-set size.

## When NOT to use

- When the goal is a one-off visualization of a fixed dataset with no new cells to project — plain fit_transform is sufficient and the parametric mapping is not being exercised
- When labels are available for the new cells and you could retrain ivis on the full data — re-embedding avoids any extrapolation error from the parametric mapping
- When features are unscaled/raw: ivis assumes inputs preprocessed to the [0, 1] range (e.g. with MinMaxScaler, or arcsinh-transformed for CyTOF data), and out-of-range features invalidate the mapping

## Inputs

- Feature matrix of single-cell (CyTOF/scRNA-seq) expression values, e.g. BMMC cells from the Levine-32-dim benchmark (104,184 cells with manual cell-type labels)
- Manually gated cell-population labels for classifier training and out-of-sample accuracy evaluation
- A previously fitted ivis model (or the subsample it was trained on)

## Outputs

- Low-dimensional embeddings of held-out cells produced via the ivis parametric transform
- Out-of-sample classification accuracy per subsample size (size/accuracy pairs, e.g. accuracy_vs_subsample_size.csv)
- Accuracy-versus-subsample-size plot (Fig. 6A analogue)

## How to apply

Fit ivis (siamese network, triplet loss with positives sampled from the KNN vector and negatives from outside the KNN vector, k=15) on a MinMaxScaler-scaled [0,1] subsample of the cells, holding out a fraction of cells. Call the fitted model's transform method to pass the held-out cells through the learned parametric mapping into the 2-dimensional embedding without retraining. Train a classifier (the source task uses a random forest; the paper's embedding-accuracy benchmark used an SVM trained on 75% and evaluated on 25%) on the subsample's embeddings and predict held-out cell labels. Repeat across stratified subsample sizes (n=1,000; 2,000; 5,000; 10,000; 30,000 from the 104,184 labeled BMMC cells) to characterize how out-of-sample accuracy scales with training-set size. Decision point: use the parametric transform rather than re-embedding whenever the question is extrapolation/generalization; re-run fit_transform only when the full data distribution changes.

## Related tools

- **ivis** (Siamese neural network trained with triplet loss (KNN-vector positives, outside-KNN-vector negatives, k=15) whose parametric mapping projects out-of-sample cells into the existing embedding via transform) — https://github.com/beringresearch/ivis
- **Python** (Implementation environment for MinMaxScaler preprocessing, ivis training/transform, and random forest classification) — https://github.com/beringresearch/ivis
- **R** (Alternative interface for running ivis and the transform-based out-of-sample projection) — https://github.com/beringresearch/ivis

## Examples

```
from sklearn.preprocessing import MinMaxScaler; from ivis import Ivis; X = MinMaxScaler().fit_transform(X); model = Ivis(embedding_dims=2, k=15).fit(X_train); emb_oos = model.transform(X_heldout)
```

## Evaluation signals

- Out-of-sample accuracy at subsample size n=1,000 is approximately 0.91 (within ±0.03) on BMMC held-out predictions
- Accuracy increases monotonically with ivis training subsample size (1,000 → 30,000) toward a plateau at large subsample sizes
- Held-out cells are embedded via model.transform with no refitting of the siamese network; embedding dimensionality equals the fitted model (2 dims)
- All input features are in the [0, 1] range after MinMaxScaler before both fit and transform
- Out-of-sample accuracy approaches the in-sample classifier accuracy at large subsample sizes, indicating the parametric mapping preserves class separability

## Limitations

- Out-of-sample accuracy degrades when the parametric mapping is trained on very small subsamples (worst run used <1% of 104,184 cells); performance is bounded by ivis training-set size
- ivis's robustness claims depend on hyperparameter settings whose full behavior is documented only via supplementary figures; the transform generalization should be validated per dataset
- The classifier-based accuracy metric reflects label separability in the embedding, not distance preservation — use Mantel-test/PCC checks on distance matrices for structure-preservation validation

## Evidence

- [abstract] adds new data points to existing embeddings using a parametric mapping function: "adds new data points to existing embeddings using a parametric mapping function"
- [methods] scaled to [0,1] before ivis: "Prior to preprocessing, values in both datasets were scaled to [0, 1] range."
- [readme] transform method for new data points: [evidence span withheld — anchor status: external_doc]
- [discussion] triplet sampling KNN positives/negatives: "ivis samples positive and negative data-points for each triplet, from the KNN vector and outside the KNN vector respectively"
- [methods] classifier-based embedding accuracy protocol: "Accuracy was calculated by training a Support Vector Machine classifier on 75% of each subsample and evaluating the classifier performance on the remaining 25%"
