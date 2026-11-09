---
name: single-cell-similarity-matrix-transformation
description: Use when a raw gene/protein expression CSV matrix (e.g. prot_expression.csv)
  plus class labels (target.csv) must be transformed before building the GraphDDP/GraphEmbed
  similarity graph.
when_to_use_negative:
- The input is already a normalized or correlation-transformed matrix (e.g. a precomputed
  similarity matrix), since re-applying --normalization or --correlation_transformation
  would distort the intended scale.
- The analysis goal is pseudotime reconstruction with tree-based methods (e.g. TSCAN's
  cluster-based MST) rather than building the GraphDDP knn/k-shift edge graph, where
  these matrix filters are not defined.
- A class has fewer than --min_threshold (default 5) instances, which the filter step
  is designed to exclude rather than transform.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GraphEmbed
  role: CLI tool whose --normalization, --correlation_transformation and --feature_selection
    options implement the matrix transformations applied before similarity-density
    and edge construction
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: graph_embed
  role: Python2/conda distribution of the same graph_embed.py program that executes
    the transformation flags on the input CSV matrix
  repo: https://github.com/fabriziocosta/GraphEmbed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:e0958227b5dc8989299eedaa559cb0fa680682cfd10794af33225ee99562198b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fabriziocosta/GraphEmbed
  tools:
  - GraphDDP
  - GraphEmbed
  - graph_embed
  - pygraphviz
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-cell-similarity-matrix-transformation

## Summary

Preprocesses the raw single-cell expression CSV matrix with GraphEmbed's --normalization (convert data matrix to normalized matrix) and --correlation_transformation (convert data matrix to correlation coefficient matrix) toggles so that subsequent cosine-similarity density estimates and knn/k-shift edge construction operate on comparable, scale-free profiles.

## When to use

Use when a raw gene/protein expression CSV matrix (e.g. prot_expression.csv) plus class labels (target.csv) must be transformed before building the GraphDDP/GraphEmbed similarity graph.

## When NOT to use

- The input is already a normalized or correlation-transformed matrix (e.g. a precomputed similarity matrix), since re-applying --normalization or --correlation_transformation would distort the intended scale.
- The analysis goal is pseudotime reconstruction with tree-based methods (e.g. TSCAN's cluster-based MST) rather than building the GraphDDP knn/k-shift edge graph, where these matrix filters are not defined.
- A class has fewer than --min_threshold (default 5) instances, which the filter step is designed to exclude rather than transform.

## Inputs

- Input data matrix in CSV format (e.g. example/prot_expression.csv), one instance per row
- Class/target assignment file in CSV format (e.g. example/target.csv), one label per instance
- Transformation flags: --normalization, --correlation_transformation, --feature_selection with --min_threshold (default 5) and --max_threshold (default 300) instances per class

## Outputs

- Transformed (normalized and/or correlation coefficient) data matrix used internally for cosine-similarity density computation
- 2D_coords.txt — the 2D embedding coordinates, one line per instance
- target.txt — the target identifier, one line per instance
- img_1_clean.pdf through img_5_hull_link.pdf images and a log file, written to a timestamped output directory (option -o)

## How to apply

Load the input data matrix (-i, CSV) and class assignments (-t, CSV) in graph_embed.py, then apply the transformation flags before density estimation: --normalization converts the data matrix to a normalized matrix, and --correlation_transformation converts it to a correlation coefficient matrix; optionally add --feature_selection to select the most discriminative features. Because GraphEmbed defines density as the sum of pairwise cosine similarity between an instance and all other instances, and k-shift edges connect each instance to a denser, different-label neighbor within the z-nearest-neighbor horizon (default z=5), these transformations directly change which neighbors are considered denser and thus which differentiation-pathway edges are drawn. Apply min/max per-class thresholds (--min_threshold=5, --max_threshold=300) so classes with too few or too many instances are handled before layout. The transformed matrix feeds the same downstream chain in both cases: outlier detection via -l mutual nearest neighbors (default 3), knn edges (-k, default 3), k-shift edges (-d, default 1), and the Kamada–Kawai force layout producing 2D coordinates.

## Related tools

- **GraphEmbed** (CLI tool whose --normalization, --correlation_transformation and --feature_selection options implement the matrix transformations applied before similarity-density and edge construction) — https://github.com/fabriziocosta/GraphEmbed
- **graph_embed** (Python2/conda distribution of the same graph_embed.py program that executes the transformation flags on the input CSV matrix) — https://github.com/fabriziocosta/GraphEmbed

## Examples

```
./graph_embed.py -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 --normalization --correlation_transformation
```

## Evaluation signals

- With --correlation_transformation, the transformed matrix entries are correlation coefficients (bounded, scale-free), which changes the density ranking 'sum of the pairwise cosine similarity between an instance and all the other instances' relative to the raw matrix.
- The output timestamped directory contains 2D_coords.txt with exactly one line per instance and target.txt with one target identifier per instance, confirming the transformation preserved instance order and count.
- The log file records the transformation flags actually applied (--normalization, --correlation_transformation, --feature_selection) for reproducibility.
- Classes with fewer than --min_threshold=5 or more than --max_threshold=300 instances are filtered before layout, so the number of nodes in img_2_edges.pdf should reflect the threshold-filtered instance set.
- The resulting img_2_edges.pdf is a valid PDF that distinguishes red knn edges from blue k-shift edges; changing the transformation flags should visibly alter which k-shift (blue) edges connect denser, different-label neighbors.

## Limitations

- The transformation options are simple binary toggles; the article/README report no quantitative metrics for choosing --normalization vs --correlation_transformation on a given dataset.
- --feature_selection and the min/max thresholds (default 5 and 300 instances per class) can silently drop classes or features, changing the graph's node set.
- The paper's discussion reports no dataset accessions or software versions for the compared tools, so transformation choices cannot be benchmarked against published parameter settings.

## Evidence

- [readme] README documents the normalization toggle: "--normalization                   Convert data matrix to normalized matrix."
- [readme] README documents the correlation transformation toggle: "--correlation_transformation      Convert data matrix to corr coeff matrix."
- [readme] Density is cosine-similarity based, so matrix transformation affects edge selection: "Density is defined as the sum of the pairwise cosine similarity between an instance and all the other instances."
- [methods] Article workflow shows the canonical invocation with input and target CSVs: "./graph_embed.py -i example/prot_expression.csv -t example/target.csv"
- [methods] Article methods list the transformation flags alongside feature selection thresholds: "--min_threshold=N                 Min num instances per class [default: 5]"
