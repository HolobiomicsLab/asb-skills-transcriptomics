---
name: symmetrized-multiclass-error-computation
description: Use when cluster-labeled single-cell profiles exist and you need to quantify,
  per cluster pair or triple (e.g. C5-C6-C7 vs C2-C3-C4), whether a transition pathway
  is supported by class-confusability rather than geometry alone.
when_to_use_negative:
- The input lacks a prior user-defined cluster assignment — GraphDDP and the confusion
  scoring start from a user-defined cluster label per cell
- The goal is per-cell pseudotime ordering rather than cluster-level transition assessment
  — pseudotime tools like TSCAN or Monocle are the wrong frame here
- The data matrix should not be converted to pairwise profiles (correlation-coefficient
  matrix is required as the first step), e.g. when features are already pairwise similarities
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: graph_embed
  role: Applies the --correlation_transformation to convert the expression matrix
    into a correlation-coefficient matrix before error estimation
  repo: github.com/fabriziocosta/GraphEmbed
- name: GraphEmbed
  role: Provides the supervised graph-embedding pipeline whose inputs (data CSV plus
    target CSV) feed the confusion-score computation
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: GraphDDP
  role: Article method that consumes cluster assignments and the confusion scores
    to visualize differentiation pathways
  repo: https://github.com/fabriziocosta/GraphEmbed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:c3873a5bc777581fff9ba7bac1f036603282f6025d4a740a2a8a454cf8e4f40c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphDDP
  - GraphEmbed
  - graph_embed
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# symmetrized-multiclass-error-computation

## Summary

Computes per-class cross-validation errors via one-vs-rest multiclass logistic regression, symmetrizes the error matrix, and derives pairwise confusion scores to assess whether a cluster triple forms a clear differentiation pathway in the GraphDDP framework.

## When to use

Use when cluster-labeled single-cell profiles exist and you need to quantify, per cluster pair or triple (e.g. C5-C6-C7 vs C2-C3-C4), whether a transition pathway is supported by class-confusability rather than geometry alone.

## When NOT to use

- The input lacks a prior user-defined cluster assignment — GraphDDP and the confusion scoring start from a user-defined cluster label per cell
- The goal is per-cell pseudotime ordering rather than cluster-level transition assessment — pseudotime tools like TSCAN or Monocle are the wrong frame here
- The data matrix should not be converted to pairwise profiles (correlation-coefficient matrix is required as the first step), e.g. when features are already pairwise similarities

## Inputs

- Expression profiles CSV (cell x feature matrix, e.g. prot_expression.csv)
- Cluster-target CSV (discrete class label per instance, e.g. target.csv)
- Cluster triples of interest (e.g. C5-C6-C7, C2-C3-C4)

## Outputs

- Symmetrized multiclass error matrix (per-class 5-fold CV error estimates)
- ternary_confusion_scores.csv with a confusion score in [0,1] per class triple
- Ternary (simplex) plot coordinates of per-class scores

## How to apply

Load the expression profiles CSV and the cluster-target CSV with pandas, then transform the data matrix into a correlation-coefficient matrix using the --correlation_transformation option of graph_embed. Fit a one-vs-rest multiclass logistic regression with the liblinear solver and evaluate it with 5-fold cross-validation to obtain per-class multiclass error estimates. Symmetrize the resulting multiclass error matrix, then compute the confusion score for each class triple as the minimum symmetrized error divided by the sum of the other two errors, yielding a value between 0 and 1. Low scores (e.g. 0.06 for the C5-C6-C7 triple) indicate a clear differentiation pathway, whereas high scores (e.g. 0.72 for C2-C3-C4) indicate confusable, poorly separated classes. Convert the three per-class scores into 3-class ternary (simplex) plot coordinates to visualize putative transitional states.

## Related tools

- **graph_embed** (Applies the --correlation_transformation to convert the expression matrix into a correlation-coefficient matrix before error estimation) — github.com/fabriziocosta/GraphEmbed
- **GraphEmbed** (Provides the supervised graph-embedding pipeline whose inputs (data CSV plus target CSV) feed the confusion-score computation) — https://github.com/fabriziocosta/GraphEmbed
- **GraphDDP** (Article method that consumes cluster assignments and the confusion scores to visualize differentiation pathways) — https://github.com/fabriziocosta/GraphEmbed

## Examples

```
./graph_embed.py -i example/prot_expression.csv -t example/target.csv --correlation_transformation --min_threshold 5 --max_threshold 300
```

## Evaluation signals

- ternary_confusion_scores.csv must reproduce 0.06 for the C5-C6-C7 triple within tolerance
- ternary_confusion_scores.csv must reproduce 0.72 for the C2-C3-C4 triple within tolerance
- All confusion scores fall in the interval [0, 1] by construction (min symmetrized error divided by the sum of the other two)
- The multiclass error matrix is symmetric after symmetrization (E[i,j] == E[j,i])
- Each per-class error comes from 5-fold cross-validation of a one-vs-rest liblinear logistic regression, not a single held-out split

## Limitations

- Class sizes are constrained: --min_threshold (default 5) and --max_threshold (default 300) instances per class in graph_embed filter classes out of the analysis
- A high confusion score (e.g. 0.72) only signals lack of a clear differentiation pathway; it does not by itself identify the correct topology
- No quantitative performance metrics are reported in the article to benchmark these error-based scores against alternative transition measures

## Evidence

- [other] a confusion score between 0 and 1 quantifies whether a clear differentiation pathway exists, e.g. 0.06 for C5-C6-C7 versus 0.72 for C2-C3-C4.: [evidence span withheld — anchor status: not_found]
- [other] Fit a one-vs-rest multiclass logistic regression with the liblinear solver and evaluate it with 5-fold cross-validation: [evidence span withheld — anchor status: not_found]
- [readme] --correlation_transformation      Convert data matrix to corr coeff matrix.: "--correlation_transformation      Convert data matrix to corr coeff matrix."
- [readme] --min_threshold=N                 Min num instances per class [default: 5]: "--min_threshold=N                 Min num instances per class [default: 5]"
