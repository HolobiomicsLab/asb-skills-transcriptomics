---
name: single-cell-cluster-visualization
description: Use when single-cell profiles (e.g. CSV expression matrix plus discrete
  class labels) must be visualized in 2D while preserving both cluster structure and
  differentiation pathways, especially when t-SNE/PCA/MDS fail to reveal trajectories.
when_to_use_negative:
- The analysis goal is pseudotime ordering alone (e.g. cluster-then-MST ordering)
  rather than a supervised 2D visualization — TSCAN is the better fit
- No discrete class/cluster labels are available, since the knn edge construction
  and class-confidence contraction both require a prior user-defined cluster assignment
- A quantitative comparison of dimensionality-reduction biases (e.g. variance explained
  by PCA) is needed; the article reports no quantitative metrics for GraphDDP vs t-SNE/PCA/MDS,
  only qualitative trajectory visibility
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GraphDDP
  role: The supervised graph-embedding method integrating cluster-membership and density-gradient
    trajectory edges for the 2D layout
  repo: github.com/fabriziocosta/GraphEmbed
- name: GraphEmbed
  role: PyPI-installable implementation of graph_embed.py that builds the knn/k-shift
    graph and computes the Kamada-Kawai embedding
  repo: github.com/fabriziocosta/GraphEmbed
- name: graph_embed
  role: Python2 conda (bioconda) distribution of the same embedding tool
  repo: github.com/fabriziocosta/GraphEmbed
- name: pygraphviz
  role: Non-pip graphviz dependency required for installation
  repo: github.com/fabriziocosta/GraphEmbed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:e8a60aed5fea647b30da2a932e457c4cad56676f6a6e9ed0cddd1bcccabe9169
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphDDP
  - GraphEmbed
  - graph_embed
  - pygraphviz
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# single-cell-cluster-visualization

## Summary

Produce a supervised 2D visualization of single-cell data that integrates cluster membership and differentiation trajectories using GraphDDP's force-based (Kamada-Kawai) graph embedding of knn same-class edges and density-gradient k-shift edges.

## When to use

Use when single-cell profiles (e.g. CSV expression matrix plus discrete class labels) must be visualized in 2D while preserving both cluster structure and differentiation pathways, especially when t-SNE/PCA/MDS fail to reveal trajectories.

## When NOT to use

- The analysis goal is pseudotime ordering alone (e.g. cluster-then-MST ordering) rather than a supervised 2D visualization — TSCAN is the better fit
- No discrete class/cluster labels are available, since the knn edge construction and class-confidence contraction both require a prior user-defined cluster assignment
- A quantitative comparison of dimensionality-reduction biases (e.g. variance explained by PCA) is needed; the article reports no quantitative metrics for GraphDDP vs t-SNE/PCA/MDS, only qualitative trajectory visibility

## Inputs

- CSV data matrix of instance features (e.g. example/prot_expression.csv)
- CSV file of discrete class labels, one per instance (e.g. example/target.csv)
- Optional preprocessing flags: --normalization, --correlation_transformation, --feature_selection

## Outputs

- 2D_coords.txt — 2D embedding coordinates, one line per instance
- target.txt — target identifier, one line per instance
- img_1_clean.pdf — image of the 2D embedding
- img_2_edges.pdf / img_3_edge.pdf — embeddings with knn (red) and k-shift (blue) edges
- img_4_hull.pdf / img_5_hull_link.pdf — embeddings with class convex hulls
- log — program execution log, written to a timestamped output directory

## How to apply

Load the expression matrix (-i, e.g. example/prot_expression.csv) and discrete class targets (-t, e.g. example/target.csv), optionally applying --normalization, --correlation_transformation, or --feature_selection. Build a graph with knn edges to the k-th nearest same-label instance (default -k 3) and k_shift edges to denser different-class instances within a -z neighbor horizon (default 5), defining density as the sum of pairwise cosine similarities and removing k-shift edges touching outliers (instances lacking -l, default 3, mutual nearest neighbors). Compute desired distances as Euclidean edge lengths, dividing distances between same-label endpoints by 1 + class_confidence (-c), then run the Kamada-Kawai force layout to obtain 2D coordinates. Choose -c according to data quality: 0.5–1 for clean, well-separated clusters and 5–30 for noisy data needing forced separation, keeping --random_state fixed to compare runs. Write the timestamped output directory containing 2D_coords.txt, target.txt, img_1_clean.pdf, and the edge/hull images and log.

## Related tools

- **GraphDDP** (The supervised graph-embedding method integrating cluster-membership and density-gradient trajectory edges for the 2D layout) — github.com/fabriziocosta/GraphEmbed
- **GraphEmbed** (PyPI-installable implementation of graph_embed.py that builds the knn/k-shift graph and computes the Kamada-Kawai embedding) — github.com/fabriziocosta/GraphEmbed
- **graph_embed** (Python2 conda (bioconda) distribution of the same embedding tool) — github.com/fabriziocosta/GraphEmbed
- **pygraphviz** (Non-pip graphviz dependency required for installation) — github.com/fabriziocosta/GraphEmbed

## Examples

```
./graph_embed.py -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 -l 0 --random_state 2 --normalization --correlation_transformation --feature_selection
```

## Evaluation signals

- Output directory contains all expected files (2D_coords.txt, target.txt, img_1_clean.pdf, log) and 2D_coords.txt/target.txt have exactly one line per input instance
- Higher -c values yield visibly more compact same-class clusters, matching the documented behavior that values higher than 0 indicate stronger belief
- Layout is stable across the recommended confidence regimes (C=0.5–1 for clean data, 5–30 for noisy data) when --random_state is fixed
- img_2_edges.pdf shows red knn edges within clusters and blue k-shift edges bridging classes, and img_4/img_5 show class convex hulls connected by shift-edge counts
- Outlier handling can be verified: k-shift edges whose endpoints have fewer than -l mutual nearest neighbors should be absent

## Limitations

- Robustness of the visualization depends on the class-confidence parameter; only the range 0.5–1 (clean) and 5–30 (noisy) is documented, with stability shown in Supplementary Fig. 1
- No quantitative performance metrics support the claimed superiority over t-SNE, PCA, or MDS — comparisons are qualitative
- Requires a prior user-defined cluster assignment; the approach inherits any discretization errors of that clustering, which can be problematic for rare cell types such as stem cells
- k-shift edge deletion removes outliers (instances with no mutual k neighbors), which may drop rare cells from the embedding

## Evidence

- [readme] Two edge types and confidence contraction: "A knn edge is an edge to the k-th nearest instance that has the same label. A k_shift edge is an edge to the k-th nearest instance that is denser and has a different label."
- [readme] Confidence effect on layout: "Values higher than 0 indicate a stronger belief and will result in more compact layouts for instances of the same class."
- [readme] Recommended parameter regimes: "Values of 0.5-1 are suitable for clean data where clusters are naturally well separated, values of 5-30 are suitable for noisy data"
- [readme] Kamada-Kawai layout algorithm: "using the force layout algorithm from Tomihisa Kamada, and Satoru Kawai. "An algorithm for drawing general undirected graphs.""
- [abstract] GraphDDP edge design rationale: "uses a force-based graph layout approach on two types of carefully constructed edges: one emphasizing cluster membership, the other, based on density gradients, emphasizing differentiation"
- [methods] Parameter documentation: "To set a desired separation strength use -c, to specify the number of neighbors use -k"
