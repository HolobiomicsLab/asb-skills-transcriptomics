---
name: pearson-correlation-feature-transformation
description: Use when preparing a scRNA-seq expression matrix (CSV) plus class labels
  for graph-embedding visualization with GraphDDP/graph_embed, after class filtering,
  normalization, and feature selection.
when_to_use_negative:
- The goal is dimensionality reduction or pseudotime ordering with tools like TSCAN,
  Monocle2, or SPRING rather than supervised force-based graph embedding
- The input is already a correlation or similarity matrix over cells; applying the
  transformation again would double-transform
- Class labels are unavailable — the pipeline (class filtering and one-vs-all SVM-SGD
  RFE) requires a discrete label per instance
tools:
- name: GraphDDP
  role: Method whose pre-processing pipeline includes the correlation transformation
    feeding the force-based graph layout
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: GraphEmbed
  role: PyPI-installable implementation exposing --correlation_transformation in graph_embed.py
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: graph_embed
  role: Python2/bioconda variant run as graph_embed with the same flags
  repo: https://github.com/fabriziocosta/GraphEmbed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:7fd9810d1a4a6c08213a288cf6938769a4a20bee6aff752640eb476e55702f75
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
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pearson-correlation-feature-transformation

## Summary

Pre-processing step in the GraphDDP/GraphEmbed pipeline that converts the feature-selected, per-cell normalized expression matrix into a Pearson correlation-coefficient matrix, mapping each cell to an R^n correlation vector used as input to the force-based graph layout.

## When to use

Use when preparing a scRNA-seq expression matrix (CSV) plus class labels for graph-embedding visualization with GraphDDP/graph_embed, after class filtering, normalization, and feature selection.

## When NOT to use

- The goal is dimensionality reduction or pseudotime ordering with tools like TSCAN, Monocle2, or SPRING rather than supervised force-based graph embedding
- The input is already a correlation or similarity matrix over cells; applying the transformation again would double-transform
- Class labels are unavailable — the pipeline (class filtering and one-vs-all SVM-SGD RFE) requires a discrete label per instance

## Inputs

- Expression data matrix in CSV format (e.g. example/prot_expression.csv), via -i
- Class/cluster label file in CSV format (e.g. example/target.csv), via -t
- Flags --normalization, --feature_selection, --min_threshold (default 5), --max_threshold (default 300)

## Outputs

- Pearson correlation-coefficient matrix over cells written to preprocessed_matrix.csv
- Downstream: 2D_coords.txt, target.txt, img_*.pdf embedding files in a timestamped output directory

## How to apply

Run graph_embed.py with --correlation_transformation together with --normalization (per-cell normalization of the expression values) and --feature_selection (RFE with a linear SVM-SGD trained one-vs-all, stopping on a 5-fold cross-validation score). Before the transformation, filter classes below the --min_threshold (default 5 instances per class) and cap classes at --max_threshold (default 300). The transformation converts the feature-selected data matrix to a Pearson correlation-coefficient matrix, so each cell is represented as an R^n vector of pairwise correlations with other cells, which is the representation on which the knn/k_shift edges (cosine-similarity density) of the force-based Kamada-Kawai layout are computed. The result is written to preprocessed_matrix.csv before the 2D embedding is produced.

## Related tools

- **GraphDDP** (Method whose pre-processing pipeline includes the correlation transformation feeding the force-based graph layout) — https://github.com/fabriziocosta/GraphEmbed
- **GraphEmbed** (PyPI-installable implementation exposing --correlation_transformation in graph_embed.py) — https://github.com/fabriziocosta/GraphEmbed
- **graph_embed** (Python2/bioconda variant run as graph_embed with the same flags) — https://github.com/fabriziocosta/GraphEmbed

## Examples

```
./graph_embed -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 -l 0 --random_state 2 --normalization --correlation_transformation --feature_selection
```

## Evaluation signals

- preprocessed_matrix.csv exists and parses as a well-formed CSV after the run
- The matrix is square and symmetric over cells, with all values within the [-1, 1] Pearson correlation range
- The preprocessing script (with --normalization, --correlation_transformation, --feature_selection) completes without errors
- The subsequent embedding run produces the expected outputs (2D_coords.txt, target.txt, img_*.pdf, log) in the timestamped output directory
- Diagonal entries of the correlation matrix equal 1 (self-correlation invariant)

## Limitations

- Feature selection via recursive elimination with a linear SVM-SGD is computationally driven by cross-validation and depends on --random_state (default 1), so results may vary with the seed
- Class filtering discards classes with fewer than --min_threshold (5) instances and caps classes at --max_threshold (300) instances, so rare cell types below the threshold are removed before transformation
- The article reports no quantitative metrics comparing GraphDDP to t-SNE, PCA, or MDS, and no dataset accessions for the intestinal epithelial and myeloid progenitor data

## Evidence

- [methods] Correlation transformation converts matrix to correlation coefficients: "--correlation_transformation      Convert data matrix to corr coeff matrix."
- [methods] Per-cell normalization precedes the transformation: "--normalization                   Convert data matrix to normalized matrix."
- [readme] README documents the combined invocation flags: "./graph_embed -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 -l 0 --random_state 2 --normalization --correlation_transformation --feature_selection"
- [methods] Class-size thresholds applied before transformation: "--min_threshold=N                 Min num instances per class [default: 5]"
- [readme] Density on which downstream edges rely is computed from pairwise similarities of transformed instances: "Density is defined as the sum of the pairwise cosine similarity between an instance and all the other instances."
