---
name: random-forest-cell-type-classification-on-embeddings
description: Use when labeled single-cell data (e.g. BMMC cells with 14 manually gated
  populations) has been embedded with ivis and you need a quantitative accuracy readout
  of the embedding, especially for held-out cells mapped with the parametric transform.
when_to_use_negative:
- Cells are unlabeled or labels are too coarse/noisy to serve as ground truth for
  supervised evaluation
- The goal is the embedding itself (structure preservation, Mantel-test distance correlation)
  rather than a downstream classification-accuracy metric
- You plan to retrain a new dimensionality-reduction model on the held-out cells —
  this skill specifically evaluates the parametric out-of-sample transform without
  retraining
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: ivis
  role: Trains the Siamese triplet-loss network (k=15) whose embeddings the classifier
    evaluates; its transform method supplies out-of-sample embeddings
  repo: https://github.com/beringresearch/ivis
- name: Python
  role: Implementation environment for MinMaxScaler preprocessing, ivis, and the scikit-learn
    random forest
- name: R
  role: Alternative ivis interface for the same train/embed/classify workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:20321856f43e7acfef01f1a7ec1822d5b1b8c60ca40e9e237427678e245bb2ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - R
  - ivis
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# random-forest-cell-type-classification-on-embeddings

## Summary

Trains a random forest classifier on low-dimensional ivis embeddings to score cell-type label recovery, including for out-of-sample cells projected via ivis's parametric transform. Used to quantify how embedding quality (e.g. on BMMC subsamples) translates into classification accuracy.

## When to use

Use when labeled single-cell data (e.g. BMMC cells with 14 manually gated populations) has been embedded with ivis and you need a quantitative accuracy readout of the embedding, especially for held-out cells mapped with the parametric transform.

## When NOT to use

- Cells are unlabeled or labels are too coarse/noisy to serve as ground truth for supervised evaluation
- The goal is the embedding itself (structure preservation, Mantel-test distance correlation) rather than a downstream classification-accuracy metric
- You plan to retrain a new dimensionality-reduction model on the held-out cells — this skill specifically evaluates the parametric out-of-sample transform without retraining

## Inputs

- Expression matrix (e.g. 32-dimensional BMMC CyTOF marker values) with cell-type labels for a subset of cells (104,184 of 265,627 labeled in Levine-32-dim)
- Min-max scaled feature matrix ([0, 1] via MinMaxScaler)
- ivis embeddings (2-dimensional) for training cells
- ivis parametric model checkpoint from which held-out cells are projected via transform

## Outputs

- Out-of-sample classification accuracy per ivis training subsample size (e.g. accuracy_vs_subsample_size.csv with size/accuracy pairs)
- Predicted cell-type labels for held-out embeddings
- Accuracy-versus-subsample-size plot (reproducing Fig. 6A)

## How to apply

Load the CyTOF/scRNA-seq expression matrix and scale features to the [0, 1] range with MinMaxScaler before embedding, since ivis expects values in this range. Train the ivis Siamese network (triplet loss with k=15, positives from the KNN vector and negatives from outside the KNN vector) on a stratified subsample (e.g. n=1,000–30,000 BMMC cells), holding out a fraction of labeled cells. Project the held-out cells through the trained model's transform method without retraining, then train a random forest on the subsample embeddings and predict labels for the held-out embeddings. Record accuracy per subsample size and check that accuracy rises toward a plateau (in the BMMC experiment, even n=1,000 — under 1% of 104,184 labeled cells — reached 0.91). The paper's own embedding-accuracy protocol used an SVM trained on 75% of each subsample and evaluated on the remaining 25%, which can be substituted as a lower-cost classifier benchmark.

## Related tools

- **ivis** (Trains the Siamese triplet-loss network (k=15) whose embeddings the classifier evaluates; its transform method supplies out-of-sample embeddings) — https://github.com/beringresearch/ivis
- **Python** (Implementation environment for MinMaxScaler preprocessing, ivis, and the scikit-learn random forest)
- **R** (Alternative ivis interface for the same train/embed/classify workflow)

## Examples

```
from ivis import Ivis; from sklearn.ensemble import RandomForestClassifier; m = Ivis(embedding_dims=2, k=15).fit(X_train); emb_t = m.transform(X_test); acc = RandomForestClassifier().fit(m.transform(X_train), y_train).score(emb_t, y_test)
```

## Evaluation signals

- Out-of-sample accuracy at n=1,000 is approximately 0.91 (within ±0.03) on BMMC data
- Accuracy increases monotonically with ivis training subsample size toward a plateau at large subsample sizes
- All features are scaled to the [0, 1] range before ivis training (X = MinMaxScaler().fit_transform(X))
- Held-out embeddings are produced by the frozen ivis transform, i.e. the Siamese network is not retrained on the test cells
- Subsample sizes cover n=1,000, 2,000, 5,000, 10,000, and 30,000 with stratified sampling over cell-type labels

## Limitations

- Accuracy depends on ivis training subsample size; small subsamples (n≈1,000) sit at the low end of the accuracy curve (~0.91) and results vary between randomized runs
- Only 39% (104,184 of 265,627) of Levine-32-dim cells have manually gated labels, limiting the labeled evaluation pool
- ivis embeddings should be judged alongside structure-preservation checks (e.g. Mantel test of distance-matrix correlation), since classifier accuracy alone does not capture global-structure fidelity
- Ivis runs on top of TensorFlow; hyperparameter sensitivity is documented only via supplementary figures rather than explicit values in the main text

## Evidence

- [methods] Accuracy was calculated by training a Support Vector Machine classifier on 75% of each subsample and evaluating the classifier performance on the remaining 25%: "Accuracy was calculated by training a Support Vector Machine classifier on 75% of each subsample and evaluating the classifier performance on the remaining 25%"
- [methods] MinMaxScaler preprocessing: "X = MinMaxScaler().fit_transform(X)"
- [readme] ivis parametric out-of-sample transform: [evidence span withheld — anchor status: external_doc]
- [discussion] triplet sampling scheme: "Given that ivis samples positive and negative data-points for each triplet, from the KNN vector and outside the KNN vector respectively"
- [readme] labeled BMMC cell counts (Levine-32-dim README): [evidence span withheld — anchor status: not_found]
