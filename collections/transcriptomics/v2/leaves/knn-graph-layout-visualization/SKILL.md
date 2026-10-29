---
name: knn-graph-layout-visualization
description: Use when you need a class-label-agnostic baseline visualization of single-cell
  expression data that mixes clusters, to contrast with class-informed graph embeddings
  like GraphDDP.
when_to_use_negative:
- The goal is trajectory/pseudotime ordering rather than a 2D layout baseline — a
  pseudotime method like TSCAN or Monocle2 is the appropriate comparison
- Data are raw reads or lack per-cell class labels, since the graph construction expects
  a discrete label per instance and a data matrix in CSV format
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: GraphEmbed
  role: Computes the 2D force-layout embedding of the knn graph from the expression
    and target CSV files
  repo: github.com/fabriziocosta/GraphEmbed
- name: pygraphviz
  role: Graphviz dependency required before installing GraphEmbed via PyPI
- name: SPRING
  role: k-nearest-neighbor visualization approach that the -k-only layout variant
    emulates as a baseline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:7ed770ab5a2675d73c4d00d139248100fe3dd5ccc778a55e46bc8d95d88bc8bf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphDDP
  - GraphEmbed
  - SPRING
  - TSCAN
  - pygraphviz
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# knn-graph-layout-visualization

## Summary

Builds SPRING-style k-nearest-neighbor force layouts on single-cell expression data, with and without a class-contraction prior, to serve as the baseline against which GraphDDP's informed 2D embedding is judged.

## When to use

Use when you need a class-label-agnostic baseline visualization of single-cell expression data that mixes clusters, to contrast with class-informed graph embeddings like GraphDDP.

## When NOT to use

- The goal is trajectory/pseudotime ordering rather than a 2D layout baseline — a pseudotime method like TSCAN or Monocle2 is the appropriate comparison
- Data are raw reads or lack per-cell class labels, since the graph construction expects a discrete label per instance and a data matrix in CSV format

## Inputs

- Expression matrix CSV (e.g. prot_expression.csv) for intestinal epithelial or myeloid progenitor cells
- Class/target label CSV (e.g. target.csv), one label per instance
- k-NN edge parameter k (3 or 7) and class_confidence c (0 for knn-only, e.g. 1.5 for contraction)

## Outputs

- 2D_coords.txt coordinate file, one line per instance
- target.txt class identifier file
- img_1_clean.pdf 2D embedding image
- img_2_edges.pdf embedding with knn edges (red) and k-shift edges (blue)
- Timestamped output directory produced by the -o option

## How to apply

Build a graph whose nodes are cells and whose edges link each instance to its k nearest neighbors (knn edges, class_confidence effectively 0), then compute a 2D force layout using the Kamada-Kawai algorithm as implemented in GraphEmbed/GraphDDP. Run the layout with -k 3 and -k 7 on the intestinal epithelial and myeloid progenitor datasets to reproduce the Supplementary Figs. 5-6 pattern in which, without the class-contraction prior, clusters mix in the 2D projection. Rerun with -k 7 plus a class-contraction prior (e.g. -c 1.5) to show how the supervised belief in class labels separates instances of the same class, since the desired edge length for same-label endpoints is divided by 1 + class_confidence. Export each embedding as an image (e.g. img_1_clean.pdf) together with its 2D_coords.txt coordinate file via the -o output directory option. The rationale is that k-NN-only layouts, like t-SNE and TSCAN baselines, preserve local structure but lose the class hierarchy and pathway information that GraphDDP recovers.

## Related tools

- **GraphEmbed** (Computes the 2D force-layout embedding of the knn graph from the expression and target CSV files) — github.com/fabriziocosta/GraphEmbed
- **pygraphviz** (Graphviz dependency required before installing GraphEmbed via PyPI)
- **SPRING** (k-nearest-neighbor visualization approach that the -k-only layout variant emulates as a baseline)

## Examples

```
conda install pygraphviz; pip install graphembed; graph_embed -i example/prot_expression.csv -t example/target.csv -k 7 -o knn_baseline
```

## Evaluation signals

- Without the class-contraction prior (class_confidence effectively 0), clusters mix in the 2D layout, reproducing the Supplementary Figs. 5-6 baseline pattern
- Adding -c (e.g. 1.5) yields visibly more compact layouts for instances of the same class, consistent with edge lengths divided by 1 + class_confidence
- Output directory contains 2D_coords.txt, target.txt, img_1_clean.pdf, img_2_edges.pdf, and a log file with a timestamped directory suffix
- One line per instance in 2D_coords.txt and target.txt, matching the number of input cells
- No quantitative metric is reported in the paper, so accept by expert review that the qualitative pattern (cluster mixing without class prior, separation with it) is reproduced

## Limitations

- No quantitative performance metrics are reported to support the claimed superiority of GraphDDP over t-SNE, PCA, or MDS
- No software versions or parameter settings for the compared tools (TSCAN, Monocle2, SPRING) are given
- k-NN-only layouts do not take class information into account, so class separation and differentiation pathways are lost in the baseline embedding

## Evidence

- [readme] A knn edge is an edge to the k-th nearest instance that has the same label.: "A knn edge is an edge to the k-th nearest instance that has the same label."
- [readme] If the endpoints of an edge have the same label then the desired distance is divided by 1 + class_confidence.: "If the endpoints of an edge have the same label then the desired distance is divided by 1 + class_confidence."
- [methods] To set a desired separation strength use -c, to specify the number of neighbors use -k: "To set a desired separation strength use -c, to specify the number of neighbors use -k"
- [intro] the classical dimensionality reduction approaches are unsupervised, e.g. they do not take into account class information available: "the classical dimensionality reduction approaches are unsupervised, e.g. they do not take into account class information available"
- [methods] The option -o allows to set a directory where the produced files are placed. The directory name is suffixed with a time stamp: "The option -o allows to set a directory where the produced files are placed. The directory name is suffixed with a time stamp"
