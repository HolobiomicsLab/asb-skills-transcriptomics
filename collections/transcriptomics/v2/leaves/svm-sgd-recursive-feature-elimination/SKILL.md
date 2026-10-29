---
name: svm-sgd-recursive-feature-elimination
description: Use when a single-cell expression matrix (CSV) with user-defined class
  labels must be reduced to discriminative features prior to graph embedding or correlation
  transformation, especially when class filtering (--min_threshold 5 / --max_threshold
  300) has already been applied.
when_to_use_negative:
- Input is already a reduced/feature-selected matrix or a correlation-coefficient
  matrix rather than a raw per-feature expression CSV.
- No class/cluster labels are available — the SVM-SGD elimination is supervised and
  requires a discrete label for each instance.
- The goal is an unsupervised embedding (PCA, MDS, t-SNE) without prior cluster information,
  which GraphDDP's class-aware pipeline is explicitly designed to complement.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GraphEmbed
  role: Implements graph_embed.py with the --feature_selection flag performing SVM-SGD
    recursive feature elimination inside the pre-processing pipeline.
  repo: github.com/fabriziocosta/GraphEmbed
- name: graph_embed
  role: Python2 conda distribution (bioconda) of the same graph_embed.py pipeline
    including --feature_selection.
- name: GraphDDP
  role: Downstream method that consumes the feature-selected, correlation-transformed
    matrix in its force-based graph layout.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:aada98c2f0418775c7a6924a195f7fa39093412bcff6ac313c19748a1ba76f03
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphDDP
  - GraphEmbed
  - graph_embed
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# svm-sgd-recursive-feature-elimination

## Summary

Recursive feature elimination step in the GraphDDP/GraphEmbed pre-processing pipeline that selects the most discriminative features from a single-cell expression matrix using a linear SVM-SGD trained one-vs-all, before converting the data to a Pearson correlation-coefficient matrix.

## When to use

Use when a single-cell expression matrix (CSV) with user-defined class labels must be reduced to discriminative features prior to graph embedding or correlation transformation, especially when class filtering (--min_threshold 5 / --max_threshold 300) has already been applied.

## When NOT to use

- Input is already a reduced/feature-selected matrix or a correlation-coefficient matrix rather than a raw per-feature expression CSV.
- No class/cluster labels are available — the SVM-SGD elimination is supervised and requires a discrete label for each instance.
- The goal is an unsupervised embedding (PCA, MDS, t-SNE) without prior cluster information, which GraphDDP's class-aware pipeline is explicitly designed to complement.

## Inputs

- Input expression data matrix in CSV format (e.g. example/prot_expression.csv)
- Class/target labels file in CSV format (e.g. example/target.csv)
- Optional flags: --normalization, --min_threshold (default 5), --max_threshold (default 300), --random_state

## Outputs

- Feature-selected data matrix passed to --correlation_transformation
- Pearson correlation-coefficient matrix written to preprocessed_matrix.csv (square, symmetric, values in [-1, 1])
- 2D embedding outputs in timestamped directory (2D_coords.txt, target.txt, img_1_clean.pdf, log)

## How to apply

Run graph_embed.py (GraphEmbed) with the --feature_selection flag on the input expression CSV (-i example/prot_expression.csv) and class labels CSV (-t example/target.csv). The selector recursively eliminates features using a linear SVM-SGD trained one-vs-all, with a 5-fold cross-validation score as the stopping rule for how many features to retain. Apply it after class filtering (min_threshold=5, max_threshold=300 instances per class) and, optionally, after --normalization, and before --correlation_transformation, which maps each cell to an R^n vector of Pearson correlation coefficients written to preprocessed_matrix.csv. The rationale is to restrict the correlation transformation to discriminative genes/features so that the downstream force-based graph layout reflects class-aware structure. Tune separation strength (-c, e.g. 0.5-1 for clean data, 5-30 for noisy data) and neighbors (-k) in the same invocation, and fix --random_state for reproducibility.

## Related tools

- **GraphEmbed** (Implements graph_embed.py with the --feature_selection flag performing SVM-SGD recursive feature elimination inside the pre-processing pipeline.) — github.com/fabriziocosta/GraphEmbed
- **graph_embed** (Python2 conda distribution (bioconda) of the same graph_embed.py pipeline including --feature_selection.)
- **GraphDDP** (Downstream method that consumes the feature-selected, correlation-transformed matrix in its force-based graph layout.)

## Examples

```
./graph_embed -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 --random_state 2 --normalization --correlation_transformation --feature_selection
```

## Evaluation signals

- preprocessed_matrix.csv exists and is a well-formed CSV after the pipeline completes.
- preprocessed_matrix.csv is square and symmetric over cells, with all values within the [-1, 1] correlation range.
- The preprocessing script (graph_embed.py with --feature_selection --correlation_transformation --normalization) runs to completion without errors.
- The 5-fold cross-validation score is used as the stopping rule, so the retained feature count should be reproducible given the same --random_state.
- Output directory contains the expected files (2D_coords.txt, target.txt, img PDFs, log) with one line per instance in the coordinate and target files.

## Limitations

- Feature selection is supervised and depends on the quality of the user-defined cluster assignment; noisy or incorrect labels will bias which features are retained.
- Article reports no quantitative performance metrics supporting GraphDDP's superiority over t-SNE, PCA, or MDS, and no explicit limitations/failure cases are discussed.
- Python2 conda version (graph_embed via bioconda) may be incompatible with modern environments; the PyPI graphembed package requires a non-pip graphviz dependency (conda install pygraphviz).

## Evidence

- [methods] SVM-SGD RFE stopping rule: [evidence span withheld — anchor status: not_found]
- [readme] feature selection option: "--feature_selection               Select most discriminative features."
- [readme] example invocation with feature selection: "./graph_embed -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 -l 0 --random_state 2 --normalization --correlation_transformation --feature_selection"
- [methods] correlation transformation mapping: [evidence span withheld — anchor status: not_found]
- [readme] supervised class expectations: "Compute a 2D embedding of a data matrix given supervised class information. A discrete label for each instance is expected."
