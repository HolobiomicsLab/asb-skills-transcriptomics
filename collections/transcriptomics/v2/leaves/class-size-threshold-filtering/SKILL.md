---
name: class-size-threshold-filtering
description: Use when a user-supplied cluster/class assignment file for GraphDDP or
  GraphEmbed contains rare or over-sized classes, and the embedding pipeline must
  drop classes with fewer than min_threshold (default 5) or cap classes at max_threshold
  (default 300) instances before layout.
when_to_use_negative:
- The class assignment is not user-supervised input but an unsupervised embedding
  task with no labels (plain t-SNE/PCA/MDS use case) — thresholds on instances per
  class are meaningless without classes
- 'Rare classes are the biological target of the analysis (e.g. stem cells): min_threshold=5
  would deliberately discard exactly the subpopulations GraphDDP was designed to help
  reveal'
- The data are already a preprocessed square correlation matrix — class-size filtering
  operates on raw cell-level expression plus labels, not on the transformed matrix
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GraphDDP
  role: Pipeline whose pre-processing stage performs the min/max instances-per-class
    filtering on the user-defined cluster assignment
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: graph_embed
  role: Python2 conda-distributed CLI exposing --min_threshold and --max_threshold
    options that implement the class-size filtering
  repo: http://conda.pydata.org/miniconda.html
- name: GraphEmbed
  role: PyPI-installed implementation of graph_embed.py whose --min_threshold=N (default
    5) and --max_threshold=N (default 300) flags control the filtering
  repo: https://github.com/fabriziocosta/GraphEmbed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:3f95a74ce37ec5b7bd6b2b8cc5a569e3d65081cc931733b41cbe90a5730c624e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fabriziocosta/GraphEmbed
  tools:
  - GraphDDP
  - GraphEmbed
  - graph_embed
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# class-size-threshold-filtering

## Summary

Filter single-cell class labels by minimum and maximum instances-per-class thresholds (defaults: 5 and 300) as the first pre-processing stage of GraphDDP/GraphEmbed, before normalization, feature selection, and correlation transformation.

## When to use

Use when a user-supplied cluster/class assignment file for GraphDDP or GraphEmbed contains rare or over-sized classes, and the embedding pipeline must drop classes with fewer than min_threshold (default 5) or cap classes at max_threshold (default 300) instances before layout.

## When NOT to use

- The class assignment is not user-supervised input but an unsupervised embedding task with no labels (plain t-SNE/PCA/MDS use case) — thresholds on instances per class are meaningless without classes
- Rare classes are the biological target of the analysis (e.g. stem cells): min_threshold=5 would deliberately discard exactly the subpopulations GraphDDP was designed to help reveal
- The data are already a preprocessed square correlation matrix — class-size filtering operates on raw cell-level expression plus labels, not on the transformed matrix

## Inputs

- Class labels CSV file (e.g. example/target.csv), one discrete label per instance
- Expression data matrix CSV file (e.g. example/prot_expression.csv), rows = cells, columns = genes/features

## Outputs

- Filtered set of classes (class labels with instance count >= min_threshold and <= max_threshold) paired with the correspondingly filtered data matrix
- Downstream preprocessed_matrix.csv (square symmetric Pearson correlation matrix over filtered cells) if subsequent pipeline stages are enabled

## How to apply

Load the class labels CSV (e.g. example/target.csv) and the expression matrix (e.g. example/prot_expression.csv) with pandas, then remove classes whose instance count falls below the minimum threshold (--min_threshold, default 5) and cap or subsample classes exceeding the maximum threshold (--max_threshold, default 300). This filtering is applied to the user-defined cluster assignment before the --normalization, --feature_selection (recursive elimination with a linear SVM-SGD, one-vs-all, 5-fold CV stopping rule), and --correlation_transformation stages run. The rationale is that classes with too few cells cannot support meaningful knn/k_shift edges in the force-based graph layout, while very large classes would dominate it. The filtered classes and the filtered data matrix jointly define the instances that enter the graph, so filtering must happen before feature selection and correlation matrix computation, not after.

## Related tools

- **GraphDDP** (Pipeline whose pre-processing stage performs the min/max instances-per-class filtering on the user-defined cluster assignment) — https://github.com/fabriziocosta/GraphEmbed
- **graph_embed** (Python2 conda-distributed CLI exposing --min_threshold and --max_threshold options that implement the class-size filtering) — http://conda.pydata.org/miniconda.html
- **GraphEmbed** (PyPI-installed implementation of graph_embed.py whose --min_threshold=N (default 5) and --max_threshold=N (default 300) flags control the filtering) — https://github.com/fabriziocosta/GraphEmbed

## Examples

```
./graph_embed.py -i example/prot_expression.csv -t example/target.csv --min_threshold=5 --max_threshold=300 --normalization --correlation_transformation --feature_selection
```

## Evaluation signals

- After filtering, every retained class has at least --min_threshold (default 5) instances and at most --max_threshold (default 300) instances
- The number of rows in the filtered expression matrix equals the number of labels retained in the filtered target file (row-label alignment invariant)
- The pipeline continues to completion: preprocessed_matrix.csv is a well-formed, square, symmetric CSV with values in [-1, 1] over the filtered cells
- graph_embed.py exits without errors and the timestamped output directory contains 2D_coords.txt, target.txt, and the img_*.pdf files

## Limitations

- Default thresholds (min 5, max 300) are fixed heuristics; the article and README do not report sensitivity analyses or justification for alternative values
- Filtering happens only when the supervised pipeline stages are invoked; the README does not document behavior when a class lies exactly at a threshold boundary
- The article reports no quantitative metrics validating the filtering step in isolation; its effect is only assessed via the final 2D embedding quality

## Evidence

- [methods] Class-size thresholds bound the labels entering the supervised graph: "--min_threshold=N                 Min num instances per class [default: 5]"
- [methods] Upper bound on class size: "--max_threshold=N                 Max num instances per class [default: 300]"
- [readme] Filtering precedes normalization and correlation transformation in the README usage example: "graph_embed -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 -l 0 --random_state 2 --normalization --correlation_transformation --feature_selection"
- [readme] Rationale: supervised edges are label-dependent, so class composition shapes the graph: "A knn edge is an edge to the k-th nearest instance that has the same label."
- [intro] Rare cell types are the motivating case, warning against over-aggressive min-threshold filtering: "This is especially true for rare cell types such as stem cells."
