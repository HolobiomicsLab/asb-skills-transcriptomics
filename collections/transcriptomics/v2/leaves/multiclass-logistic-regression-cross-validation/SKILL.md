---
name: multiclass-logistic-regression-cross-validation
description: Use when clusters from single-cell expression data must be tested for
  transitional relationships — e.g. estimating per-cell class probabilities and pairwise
  confusion scores to build ternary plots between putatively connected classes.
when_to_use_negative:
- When the goal is visualization of the embedding itself rather than testing transitional
  relationships between classes — plain graph_embed without the cross-validation/confusion-score
  analysis suffices.
- When classes are so few (< 3) that no class triple exists to score in a ternary
  plot.
- When the data matrix has already been correlation-transformed downstream, applying
  --correlation_transformation again would double-transform the profiles.
tools:
- name: graph_embed
  role: Provides the correlation_transformation option that converts the data matrix
    into a correlation-coefficient matrix before classification
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: GraphEmbed
  role: Supervised 2D embedding framework whose class-information input and profile/target
    CSV interface this analysis builds upon
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: GraphDDP
  role: The overall differentiation-pathway detection approach into which the ternary
    confusion-score analysis feeds
  repo: https://github.com/fabriziocosta/GraphEmbed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:31f2d1a784276500d7dd912186d637e59afc9a86c4371af29e35e5b96f6592a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fabriziocosta/GraphEmbed
  tools:
  - GraphDDP
  - GraphEmbed
  - graph_embed
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# Multiclass logistic regression with cross-validated error for confusion-score computation

## Summary

Fits a one-vs-rest multiclass logistic regression (liblinear solver) on cell/cluster profiles and uses 5-fold cross-validated multiclass error estimates, symmetrized per class pair, to compute confusion scores for every cluster triple. These scores (0–1) drive ternary (simplex) plots that reveal whether a cluster represents a transitional state in a differentiation pathway.

## When to use

Use when clusters from single-cell expression data must be tested for transitional relationships — e.g. estimating per-cell class probabilities and pairwise confusion scores to build ternary plots between putatively connected classes.

## When NOT to use

- When the goal is visualization of the embedding itself rather than testing transitional relationships between classes — plain graph_embed without the cross-validation/confusion-score analysis suffices.
- When classes are so few (< 3) that no class triple exists to score in a ternary plot.
- When the data matrix has already been correlation-transformed downstream, applying --correlation_transformation again would double-transform the profiles.

## Inputs

- Expression/profiles CSV matrix (rows = cells or cluster profiles, columns = features, e.g. example/prot_expression.csv)
- Cluster-target CSV assigning a class label to each instance (e.g. example/target.csv)
- Optionally, the correlation-coefficient-transformed data matrix produced by graph_embed --correlation_transformation

## Outputs

- 5-fold cross-validated multiclass error estimates per class pair (symmetrized error matrix)
- Confusion score per class triple (value between 0 and 1)
- Ternary (simplex) plot coordinates and ternary_confusion_scores.csv
- Ternary plots of estimated per-cell class probabilities for putative transition triples

## How to apply

Load the expression profiles CSV (e.g. prot_expression.csv) and cluster-target CSV with pandas, and transform the data matrix into a correlation-coefficient matrix using the --correlation_transformation option of graph_embed. Fit a one-vs-rest multiclass logistic regression with the liblinear solver and evaluate it with 5-fold cross-validation to obtain multiclass error estimates per class pair. Symmetrize the resulting multiclass error matrix, then compute the confusion score for each class triple as the minimum symmetrized error divided by the sum of the other two errors. Convert the three per-class scores into 3-class ternary (simplex) plot coordinates to visualize the estimated class probabilities. Rationale: pairs with a low minimum symmetrized error relative to the other two pairs indicate a clear differentiation pathway through the middle cluster, whereas values near 1 indicate poor separation among the triple.

## Related tools

- **graph_embed** (Provides the correlation_transformation option that converts the data matrix into a correlation-coefficient matrix before classification) — https://github.com/fabriziocosta/GraphEmbed
- **GraphEmbed** (Supervised 2D embedding framework whose class-information input and profile/target CSV interface this analysis builds upon) — https://github.com/fabriziocosta/GraphEmbed
- **GraphDDP** (The overall differentiation-pathway detection approach into which the ternary confusion-score analysis feeds) — https://github.com/fabriziocosta/GraphEmbed

## Examples

```
from sklearn.linear_model import LogisticRegression; from sklearn.model_selection import cross_val_predict; clf = LogisticRegression(solver='liblinear'); probs = cross_val_predict(clf, X_corr, y, cv=5, method='predict_proba')
```

## Evaluation signals

- Reproducing ternary_confusion_scores.csv must yield 0.06 for the C5-C6-C7 triple and 0.72 for the C2-C3-C4 triple within tolerance
- All computed confusion scores must fall in the range 0 to 1
- The multiclass error matrix must be symmetric after symmetrization (E[i][j] == E[j][i])
- Each confusion score must equal the minimum symmetrized error of the triple divided by the sum of the other two errors
- Cross-validation must be stratified and complete: 5 folds, each instance scored exactly once per fold evaluation

## Limitations

- Reported example confusion scores (0.06 for C5-C6-C7, 0.72 for C2-C3-C4) depend on the specific intestinal epithelial dataset and target assignments; other datasets need not reproduce these values
- No explicit software versions or parameter settings for the classification beyond the liblinear solver and 5-fold cross-validation are documented
- Instances below the min per-class threshold (default --min_threshold=5) or above the max (default --max_threshold=300) are excluded by the graph_embed pipeline, which can affect class error estimates

## Evidence

- [methods] correlation transform: "--correlation_transformation      Convert data matrix to corr coeff matrix."
- [readme] graph_embed invocation: "./graph_embed -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 -l 0 --random_state 2 --normalization --correlation_transformation --feature_selection"
- [methods] thresholds: "--min_threshold=N                 Min num instances per class [default: 5]"
- [abstract] tool role: "uses a force-based graph layout approach on two types of carefully constructed edges: one emphasizing cluster membership, the other, based on density gradients, emphasizing differentiation"
