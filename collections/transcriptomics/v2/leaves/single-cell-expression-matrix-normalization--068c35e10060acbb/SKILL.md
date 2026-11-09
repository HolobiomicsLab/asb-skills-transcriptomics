---
name: single-cell-expression-matrix-normalization
description: Use when a raw CSV single-cell expression matrix (e.g. prot_expression.csv)
  with a class-labels CSV must be normalized per-cell before computing a correlation-coefficient
  matrix for graph-embedding layout.
when_to_use_negative:
- Input is already a normalized matrix or a correlation-coefficient matrix (re-normalizing
  would double-transform the data).
- No supervised class labels are available and the goal is purely unsupervised dimensionality
  reduction — classical approaches like PCA, MDS, or t-SNE are the alternative the
  article contrasts against.
- The goal is pseudotime ordering rather than informed 2D visualization — tools like
  TSCAN or Monocle use clustering plus MST instead.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: GraphEmbed
  role: Python package whose graph_embed.py CLI performs the normalization, class
    filtering, feature selection, and correlation transformation before the force-based
    layout
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: graph_embed
  role: Python2/bioconda variant of the same pre-processing and embedding pipeline
    installed via conda
  repo: https://github.com/fabriziocosta/GraphEmbed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:88bed4dc61a36fe394399067dde0996a91f66cb1fc80286a5732f0d17017ed97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fabriziocosta/GraphEmbed
  tools:
  - GraphDDP
  - GraphEmbed
  - graph_embed
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-expression-matrix-normalization

## Summary

Normalize per-cell expression values of a single-cell expression matrix as part of the GraphDDP/GraphEmbed pre-processing stage, so that downstream class filtering, RFE feature selection, and Pearson correlation transformation operate on comparable cell profiles.

## When to use

Use when a raw CSV single-cell expression matrix (e.g. prot_expression.csv) with a class-labels CSV must be normalized per-cell before computing a correlation-coefficient matrix for graph-embedding layout.

## When NOT to use

- Input is already a normalized matrix or a correlation-coefficient matrix (re-normalizing would double-transform the data).
- No supervised class labels are available and the goal is purely unsupervised dimensionality reduction — classical approaches like PCA, MDS, or t-SNE are the alternative the article contrasts against.
- The goal is pseudotime ordering rather than informed 2D visualization — tools like TSCAN or Monocle use clustering plus MST instead.

## Inputs

- CSV expression matrix of single-cell profiles (e.g. example/prot_expression.csv)
- CSV class-labels/target file with a discrete label per instance (e.g. example/target.csv)
- Optional thresholds: --min_threshold (default 5), --max_threshold (default 300)

## Outputs

- Normalized expression matrix (converted data matrix)
- Preprocessed Pearson correlation-coefficient matrix written to preprocessed_matrix.csv
- Feature-selected matrix if --feature_selection is enabled

## How to apply

Load the input CSV expression matrix and class-labels CSV with pandas, then first filter classes by instance count: drop classes below --min_threshold (default 5) and cap classes at --max_threshold (default 300). Apply --normalization to convert the data matrix to a normalized matrix, normalizing per-cell expression values so downstream similarity computations are not dominated by library-size or scale differences. Optionally continue with --feature_selection, which recursively eliminates features using a linear SVM-SGD trained one-vs-all with a 5-fold cross-validation score as the stopping rule, and --correlation_transformation, which converts the matrix to a Pearson correlation-coefficient matrix mapping each cell to an R^n vector. The preprocessed correlation matrix is written to preprocessed_matrix.csv and feeds the force-based graph layout in graph_embed.py.

## Related tools

- **GraphEmbed** (Python package whose graph_embed.py CLI performs the normalization, class filtering, feature selection, and correlation transformation before the force-based layout) — https://github.com/fabriziocosta/GraphEmbed
- **graph_embed** (Python2/bioconda variant of the same pre-processing and embedding pipeline installed via conda) — https://github.com/fabriziocosta/GraphEmbed

## Examples

```
./graph_embed.py -i example/prot_expression.csv -t example/target.csv --normalization --feature_selection --correlation_transformation --min_threshold 5 --max_threshold 300
```

## Evaluation signals

- preprocessed_matrix.csv exists and is a well-formed CSV after the pipeline runs without errors.
- preprocessed_matrix.csv has square symmetric structure over cells (one row/column per cell).
- All entries of preprocessed_matrix.csv lie in the [-1, 1] Pearson correlation range.
- Classes below --min_threshold (5) are absent and classes are capped at --max_threshold (300) instances.
- With --feature_selection, only the most discriminative features (selected by one-vs-all linear SVM-SGD with 5-fold CV stopping rule) remain in the matrix.

## Limitations

- The pipeline expects a discrete label for each instance; without user-defined class assignments the normalization stage cannot feed the supervised edge construction.
- Class filtering removes rare cell types below 5 instances per class, which conflicts with the article's own point that rare cell types such as stem cells are of special interest.
- Feature selection via recursive elimination with 5-fold CV is stochastic without fixing --random_state.

## Evidence

- [abstract] GraphDDP starts from a user-defined cluster assignment and then uses a force-based graph layout approach: "GraphDDP starts from a user-defined cluster assignment and then uses a force-based graph layout approach"
- [readme] normalization flag: "--normalization                   Convert data matrix to normalized matrix."
- [readme] correlation transformation flag: "--correlation_transformation      Convert data matrix to corr coeff matrix."
- [readme] threshold options: "--min_threshold=N                 Min num instances per class [default: 5]"
- [methods] supervised usage example: "./graph_embed -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 -l 0 --random_state 2 --normalization --correlation_transformation --feature_selection"
