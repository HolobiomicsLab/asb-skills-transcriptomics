---
name: ternary-simplex-coordinate-mapping
description: Use when you have a discrete cluster-target label per cell (CSV) and
  want to test whether one of three specific clusters is transitional between the
  other two, by converting per-class errors into simplex coordinates and a confusion
  score.
when_to_use_negative:
- The question involves more or fewer than three clusters at a time — the confusion
  score is defined only per class triple
- No prior discrete cluster assignment exists for the cells (the workflow is supervised
  and starts from user-defined class labels)
- The goal is pseudotime ordering of individual cells rather than testing transitional
  relationships between three specific clusters
tools:
- name: graph_embed
  role: Applies --correlation_transformation to convert the data matrix into a correlation-coefficient
    matrix before logistic regression
  repo: github.com/fabriziocosta/GraphEmbed
- name: GraphEmbed
  role: Supervised graph-embedding framework whose input/target CSV workflow and filter
    steps (correlation_transformation, min/max_threshold) feed the triple-wise error
    estimation
  repo: github.com/fabriziocosta/GraphEmbed
- name: scikit-learn LogisticRegression
  role: One-vs-rest multiclass classifier with liblinear solver, evaluated with 5-fold
    cross-validation to produce per-class error estimates used in the confusion score
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:4c278f9231d2a68d153177eaf061de561f7bae1143447b2e1a4a4d66f46b52c6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphDDP
  - GraphEmbed
  - graph_embed
  - scikit-learn LogisticRegression
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ternary-simplex-coordinate-mapping

## Summary

Maps three putatively related clusters into ternary (simplex) plot coordinates from one-vs-rest logistic-regression class probabilities and cross-validation error estimates, and computes a confusion score (0–1) per class triple to test whether one cluster is a transitional state between the other two (e.g. 0.06 for C5-C6-C7 vs 0.72 for C2-C3-C4).

## When to use

Use when you have a discrete cluster-target label per cell (CSV) and want to test whether one of three specific clusters is transitional between the other two, by converting per-class errors into simplex coordinates and a confusion score.

## When NOT to use

- The question involves more or fewer than three clusters at a time — the confusion score is defined only per class triple
- No prior discrete cluster assignment exists for the cells (the workflow is supervised and starts from user-defined class labels)
- The goal is pseudotime ordering of individual cells rather than testing transitional relationships between three specific clusters

## Inputs

- Expression/cell profile data matrix in CSV format (e.g. example/prot_expression.csv)
- Cluster-target CSV assigning a discrete class label to each instance (e.g. example/target.csv)
- Pairwise class (multiclass error) matrix after correlation transformation of the profiles

## Outputs

- ternary_confusion_scores.csv with one confusion score per class triple (expected 0.06 for C5-C6-C7, 0.72 for C2-C3-C4)
- 3-class ternary (simplex) plot coordinates derived from the three per-class scores
- Ternary plots of estimated class probabilities per cell

## How to apply

Load the expression profiles CSV and cluster-target CSV with pandas, then apply the --correlation_transformation option of graph_embed to convert the data matrix into a correlation-coefficient matrix. Fit a one-vs-rest multiclass logistic regression with the liblinear solver and evaluate it with 5-fold cross-validation to obtain multiclass error estimates per class. Symmetrize the resulting multiclass error matrix and compute the confusion score for each class triple as the minimum symmetrized error divided by the sum of the other two errors. Finally convert the three per-class scores into 3-class ternary (simplex) coordinates; a score near 0 (e.g. 0.06 for C5-C6-C7) indicates a clear differentiation pathway, while a higher score (e.g. 0.72 for C2-C3-C4) indicates no clear transition.

## Related tools

- **graph_embed** (Applies --correlation_transformation to convert the data matrix into a correlation-coefficient matrix before logistic regression) — github.com/fabriziocosta/GraphEmbed
- **GraphEmbed** (Supervised graph-embedding framework whose input/target CSV workflow and filter steps (correlation_transformation, min/max_threshold) feed the triple-wise error estimation) — github.com/fabriziocosta/GraphEmbed
- **scikit-learn LogisticRegression** (One-vs-rest multiclass classifier with liblinear solver, evaluated with 5-fold cross-validation to produce per-class error estimates used in the confusion score)

## Examples

```
from sklearn.linear_model import LogisticRegression; from sklearn.model_selection import cross_val_score; err = 1 - cross_val_score(LogisticRegression(solver='liblinear'), X_profiles, y_clusters, cv=5).mean(); score = min_err / (err_i + err_j)
```

## Evaluation signals

- ternary_confusion_scores.csv reproduces the reported confusion score of 0.06 for the C5-C6-C7 triple within tolerance
- ternary_confusion_scores.csv reproduces the reported confusion score of 0.72 for the C2-C3-C4 triple within tolerance
- All confusion scores fall within the [0, 1] interval as defined by the min-error / sum-of-other-errors formula
- The multiclass error matrix is symmetrized before computing scores, and errors come from 5-fold cross-validation with the liblinear one-vs-rest logistic regression

## Limitations

- Confusion scores are only defined for class triples, so the skill cannot directly rank global transition structure across many clusters
- The article reports no quantitative performance metrics supporting GraphDDP's superiority over t-SNE, PCA, or MDS, and no explicit failure cases are discussed
- No dataset accessions are cited for the intestinal epithelial and myeloid progenitor data, making exact reproduction of the reported scores dependent on obtaining the original profiles

## Evidence

- [other] confusion score rationale: [evidence span withheld — anchor status: not_found]
- [other] logistic regression workflow: [evidence span withheld — anchor status: not_found]
- [readme] correlation transformation option: "--correlation_transformation      Convert data matrix to corr coeff matrix."
- [methods] target CSV input: "Specify classes data file in CSV format."
