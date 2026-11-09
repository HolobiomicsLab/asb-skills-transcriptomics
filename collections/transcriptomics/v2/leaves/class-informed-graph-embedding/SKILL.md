---
name: class-informed-graph-embedding
description: Use when you have a single-cell expression matrix (e.g. intestinal epithelial
  or myeloid progenitor cells) plus class labels from a prior clustering, and need
  a 2D visualization preserving both cluster separation and differentiation pathways.
when_to_use_negative:
- No discrete class labels or prior clustering are available — the knn edges and class_confidence
  contraction require per-instance labels
- You need a quantitative benchmark of trajectory recovery — the GraphDDP paper reports
  no quantitative metrics, only qualitative expert-review comparison against t-SNE/PCA/MDS/TSCAN/SPRING
- Classes exceed --max_threshold (default 300) instances per class or fall below --min_threshold
  (default 5) — instances are filtered before embedding
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: GraphEmbed
  role: Computes the class-informed 2D graph embedding (knn + k_shift edges, Kamada–Kawai
    force layout) from expression and target CSVs; installable via PyPI or conda (bioconda
    python2 version)
  repo: github.com/fabriziocosta/GraphEmbed
- name: pygraphviz
  role: Non-pip graphviz dependency that must be installed via conda before 'pip install
    graphembed'
- name: TSCAN
  role: 'Pseudotime baseline: cluster-based MST ordering run with 6 clusters (intestinal)
    and 20 clusters (myeloid) to show it detects only a single linear pathway and
    splits classes'
  repo: zji90/TSCAN
- name: SPRING
  role: k-NN baseline reproduced via GraphEmbed with -k 3 / -k 7 and class_confidence
    effectively 0 (no -c class-contraction prior)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:e1ed8efb12d23cf665c59d63be97549a0c741117ab9b8b899d8a020ba3720968
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphDDP
  - GraphEmbed
  - SPRING
  - TSCAN
  - pygraphviz
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# class-informed-graph-embedding

## Summary

Builds a 2D embedding of a single-cell expression matrix that integrates a prior cluster assignment with differentiation structure: a graph with same-label k-NN edges and density-gradient k-shift edges is laid out with the Kamada–Kawai force algorithm (GraphDDP/GraphEmbed).

## When to use

Use when you have a single-cell expression matrix (e.g. intestinal epithelial or myeloid progenitor cells) plus class labels from a prior clustering, and need a 2D visualization preserving both cluster separation and differentiation pathways.

## When NOT to use

- No discrete class labels or prior clustering are available — the knn edges and class_confidence contraction require per-instance labels
- You need a quantitative benchmark of trajectory recovery — the GraphDDP paper reports no quantitative metrics, only qualitative expert-review comparison against t-SNE/PCA/MDS/TSCAN/SPRING
- Classes exceed --max_threshold (default 300) instances per class or fall below --min_threshold (default 5) — instances are filtered before embedding

## Inputs

- Expression matrix in CSV format (e.g. example/prot_expression.csv)
- Class/target label CSV with a discrete label per instance (e.g. example/target.csv)
- User-defined cluster assignment (prior class knowledge)
- Parameter choices: -k neighbors, -c class_confidence, -d shift edges, -z denser-neighbor horizon, -l outlier threshold

## Outputs

- 2D_coords.txt (one line of 2D coordinates per instance)
- target.txt (one target identifier per instance)
- img_1_clean.pdf through img_5_hull_link.pdf (clean embedding, knn/k-shift edge overlays, class convex hulls)
- log file in a timestamped output directory set via -o

## How to apply

Install GraphEmbed via 'pip install graphembed' after satisfying the graphviz dependency ('conda install pygraphviz'), then run graph_embed.py with the expression matrix (-i) and class/target CSV (-t). The graph uses two edge types: 'knn' edges to the k-th nearest same-label instance (-k, default 3) and 'k_shift' edges to denser different-label instances (density = sum of pairwise cosine similarities), where a same-label edge's desired length is divided by 1 + class_confidence (-c, default 1.0; 0.5–1 for clean data, 5–30 for noisy data; -c 0 reproduces SPRING-style k-NN-only layouts that mix clusters). Nodes without mutual k neighbors are deleted as outliers (-l controls the mutual-neighbor threshold), and the 2D coordinates come from the Kamada–Kawai force layout. For the GraphDDP baseline comparison, additionally run t-SNE with elevated perplexity, PCA, MDS, and TSCAN (6 clusters on intestinal data, 20 on myeloid data) to show what class-agnostic or pseudotime-only approaches miss.

## Related tools

- **GraphEmbed** (Computes the class-informed 2D graph embedding (knn + k_shift edges, Kamada–Kawai force layout) from expression and target CSVs; installable via PyPI or conda (bioconda python2 version)) — github.com/fabriziocosta/GraphEmbed
- **pygraphviz** (Non-pip graphviz dependency that must be installed via conda before 'pip install graphembed')
- **TSCAN** (Pseudotime baseline: cluster-based MST ordering run with 6 clusters (intestinal) and 20 clusters (myeloid) to show it detects only a single linear pathway and splits classes) — zji90/TSCAN
- **SPRING** (k-NN baseline reproduced via GraphEmbed with -k 3 / -k 7 and class_confidence effectively 0 (no -c class-contraction prior))

## Examples

```
./graph_embed -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 -l 0 --random_state 2 --normalization --correlation_transformation --feature_selection
```

## Evaluation signals

- Output directory contains 2D_coords.txt with exactly one line per instance, aligned with target.txt, plus img_1_clean.pdf and the edge/hull overlays
- With a class-contraction prior (e.g. -c 1.5), instances of the same class form compact, separated groups; without it (-c 0, k-NN-only), clusters mix as expected for SPRING-style layouts
- t-SNE at elevated perplexity loses class separation, and TSCAN given the same number of classes as GraphDDP (20 on myeloid data) detects only a single linear pathway with large pseudotime spread within clusters and class splitting
- k-shift edges (blue in img_2_edges.pdf/img_3_edge.pdf) connect denser different-label instances and terminate at outlier nodes lacking mutual k neighbors
- Since no quantitative metric is reported in the paper, accept reproduction by expert review of the qualitative baseline patterns (Supplementary Figs. 2, 4–8 analogues)

## Limitations

- No quantitative performance metrics are reported to support claimed superiority over t-SNE, PCA, or MDS; evaluation is qualitative
- No software versions or parameter settings for compared tools (TSCAN, Monocle2, SPRING) and no dataset accessions are given for the intestinal epithelial and myeloid progenitor data
- k-shift edges are deleted if either endpoint is an outlier (no mutual k neighbors), so sparse subpopulations may lose trajectory links; the class label discretization does not reflect the continuous nature of differentiation

## Evidence

- [abstract] GraphDDP uses a force-based graph layout on cluster-membership and density-gradient trajectory edges: "uses a force-based graph layout approach on two types of carefully constructed edges: one emphasizing cluster membership, the other, based on density gradients, emphasizing differentiation"
- [methods] GraphEmbed installation via PyPI with pygraphviz dependency: "GraphEmbed can be installed via PyPI"
- [discussion] Pseudotime and k-NN baselines compared in the discussion: "including pseudotime-based methods like TSCAN and Monocle2, as well as k-nearest neighbor approaches like SPRING"
- [readme] Definition of knn edges in GraphEmbed README: "A knn edge is an edge to the k-th nearest instance that has the same label."
- [readme] Density definition underlying k-shift edges: "Density is defined as the sum of the pairwise cosine similarity between an instance and all the other instances."
- [readme] Output directory and timestamped files: "The option -o allows to set a directory where the produced files are placed. The directory name is suffixed with a time stamp"
