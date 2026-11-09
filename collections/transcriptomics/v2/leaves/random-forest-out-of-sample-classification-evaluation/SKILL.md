---
name: random-forest-out-of-sample-classification-evaluation
description: Use when you need a label-based, out-of-sample measure of embedding quality
  — e.g. scoring ivis embeddings of BMMC CyTOF subsamples with manually gated cell-population
  labels via held-out classification accuracy.
when_to_use_negative:
- When the goal is to assess embedding fidelity geometrically (distance preservation
  via Mantel test / Pearson correlation of distance matrices) rather than label-predictive
  quality — the article uses the Mantel-test protocol for that purpose
- When cell labels are unavailable or mostly unassigned — 61% of BMMC cells carry
  no manually gated population label, so no supervised classifier can be trained on
  them
- When evaluating in-sample accuracy on the cells the ivis model was trained on —
  this inflates scores and defeats the out-of-sample evaluation the parametric transform
  enables
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0092
tools:
- name: ivis
  role: Trains the siamese-network triplet-loss embeddings (k=15, 'maaten' baseline
    model) whose quality is being scored, and provides the transform method for projecting
    held-out cells through the parametric mapping
  repo: https://github.com/beringresearch/ivis
- name: Python
  role: Hosts the ivis library and the random-forest classifier training/evaluation
    loop over subsamples and loss/metric combinations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:35d689223362844420a7cfbcc446aee3367aad48577b2482ead062c6fd68fdae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - ivis
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# random-forest-out-of-sample-classification-evaluation

## Summary

Evaluates the quality of ivis siamese-network embeddings by training a random forest classifier on embeddings of labeled cells and scoring accuracy on held-out cells transformed through the parametric mapping. On BMMC CyTOF subsamples this distinguishes loss-function/distance-metric combinations (e.g. Euclidean pn loss reaching 0.94–0.97 accuracy).

## When to use

Use when you need a label-based, out-of-sample measure of embedding quality — e.g. scoring ivis embeddings of BMMC CyTOF subsamples with manually gated cell-population labels via held-out classification accuracy.

## When NOT to use

- When the goal is to assess embedding fidelity geometrically (distance preservation via Mantel test / Pearson correlation of distance matrices) rather than label-predictive quality — the article uses the Mantel-test protocol for that purpose
- When cell labels are unavailable or mostly unassigned — 61% of BMMC cells carry no manually gated population label, so no supervised classifier can be trained on them
- When evaluating in-sample accuracy on the cells the ivis model was trained on — this inflates scores and defeats the out-of-sample evaluation the parametric transform enables

## Inputs

- Preprocessed BMMC CyTOF expression matrix (32 surface markers, arcsinh-transformed X/5 and MinMaxScaler-scaled to [0,1])
- Manually gated cell-population labels (k=14 populations; available for 39% / 104,184 of 265,627 cells)
- Stratified random subsamples of 1,000, 5,000, 10,000, and 15,000 cells
- ivis embeddings from training subsets, plus out-of-sample embeddings of held-out cells obtained via the trained parametric mapping

## Outputs

- 8x4 accuracy table (loss-function/distance-metric combination x subsample size), e.g. Euclidean pn loss: 0.94, 0.96, 0.97, 0.97 for 1k/5k/10k/15k cells
- Per-combination out-of-sample classification accuracy scores identifying the best loss/metric combination

## How to apply

Generate stratified random subsamples of the BMMC CyTOF data (n = 1,000, 5,000, 10,000, 15,000 cells) after MinMaxScaler scaling to [0, 1]. For each subsample, train ivis embeddings with k=15 and the 'maaten' baseline model (the article's recommended setting for datasets with fewer than 500,000 observations) under each of the eight loss/distance combinations — pn loss, conventional triplet, and Softmax Ratio losses crossed with Euclidean, Manhattan, and Chebyshev metrics in the triplet-loss objective. Transform held-out cells through each trained parametric mapping (ivis transform method) to obtain out-of-sample embeddings rather than re-running the algorithm. Train a random forest classifier on the training-subset embeddings and compute classification accuracy on the held-out subset only, assembling the 8x4 loss-by-subsample-size accuracy table; expect the paper's reported pattern with Euclidean pn loss at approximately 0.94, 0.96, 0.97, and 0.97 across the four subsample sizes.

## Related tools

- **ivis** (Trains the siamese-network triplet-loss embeddings (k=15, 'maaten' baseline model) whose quality is being scored, and provides the transform method for projecting held-out cells through the parametric mapping) — https://github.com/beringresearch/ivis
- **Python** (Hosts the ivis library and the random-forest classifier training/evaluation loop over subsamples and loss/metric combinations)

## Examples

```
from sklearn.ensemble import RandomForestClassifier; acc = RandomForestClassifier().fit(train_emb, y_train).score(test_emb, y_test)
```

## Evaluation signals

- Reproduced 8x4 table matches the reported accuracy pattern, with Euclidean pn loss at approximately 0.94, 0.96, 0.97, and 0.97 for subsamples of 1,000, 5,000, 10,000, and 15,000 cells
- Accuracy is computed strictly on held-out cells transformed through the trained ivis parametric mapping, never on cells used for fit_transform
- Accuracy is non-decreasing (0.94 → 0.96 → 0.97 → 0.97) as subsample size increases, consistent with the paper's reported trend
- All eight loss/distance-metric combinations (pn, triplet, softmax-ratio x Euclidean, Manhattan, Chebyshev) are evaluated per subsample size before ranking
- Only cells with manually gated population labels (39% of BMMC cells, k=14 populations) enter classifier training and evaluation

## Limitations

- Manually gated labels cover only 39% (104,184) of the 265,627 BMMC cells; the 61% unassigned cells cannot contribute to classifier training or accuracy estimation
- The article's published embedding-accuracy protocol trained a Support Vector Machine classifier on a 75/25 subsample split; the random-forest variant follows the task-card workflow, so results may not be directly comparable to the SVM-based numbers
- Results are specific to the ivis settings used (k=15, 'maaten' baseline model recommended for datasets <500,000 observations); other ivis hyperparameter settings are not covered by this evaluation

## Evidence

- [methods] Accuracy was calculated by training a Support Vector Machine classifier on 75% of each subsample and evaluating the classifier performance on the remaining 25%: "Accuracy was calculated by training a Support Vector Machine classifier on 75% of each subsample and evaluating the classifier performance on the remaining 25%"
- [methods] Random stratified subsamples (n=50) of 1000 observations were used to generate a continuum of PCC values for each embedding technique.: "Random stratified subsamples (n=50) of 1000 observations were used to generate a continuum of PCC values for each embedding technique."
- [readme] ivis supports addition of new data points to original embeddings via a `transform` method, making it easy to incorporate ivis into standard sklearn Pipelines.: [evidence span withheld — anchor status: external_doc]
- [readme] Manually gated cell population (cluster) labels for `k = 14` major immune cell populations are available for 39% (104,184) of the cells: [evidence span withheld — anchor status: not_found]
