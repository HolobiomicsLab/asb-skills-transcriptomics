---
name: differentiation-trajectory-pathway-interpretation
description: Use when you have scRNA-seq or protein expression profiles with prior
  RaceID2/cluster class labels and need a 2D visualization that exposes differentiation
  pathways and lineage relations (e.g. stem cell clusters, branch points) beyond what
  unsupervised t-SNE/PCA/MDS shows.
when_to_use_negative:
- You have no prior class/cluster assignment for the cells — GraphDDP starts from
  a user-defined cluster assignment and is not an unsupervised embedding
- You need pseudo-time ordering or quantitative trajectory scores — pseudotime-based
  methods like TSCAN or Monocle2 address that; GraphDDP is a visualization of hierarchy
  and pathways
- Classes are smaller than the min_threshold of 5 or larger than the max_threshold
  of 300 instances per class (tool defaults may drop classes)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: GraphDDP
  role: The supervised graph-embedding visualization approach this skill implements,
    integrating cluster-based and trajectory-based viewpoints
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: GraphEmbed
  role: PyPI/conda-installable implementation of graph_embed.py that computes the
    2D embedding from input and target CSVs
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: pygraphviz
  role: Non-pip graphviz dependency to install via conda before running GraphEmbed
- name: TSCAN
  role: 'Comparison tool: cluster-based MST pseudotime method whose discretized cluster
    view GraphDDP complements; not executed by this skill'
  repo: https://github.com/zji90/TSCAN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:e4d56c7c6ece434e16f816444a76e10f8e865e1e587d7449e125d77d119193bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fabriziocosta/GraphEmbed
  tools:
  - GraphDDP
  - GraphEmbed
  - TSCAN
  - pygraphviz
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# differentiation-trajectory-pathway-interpretation

## Summary

A supervised graph-embedding skill (GraphDDP, via GraphEmbed) that visualizes single-cell transcriptome data as 2D layouts in which both cluster membership and density-gradient-driven differentiation trajectories are visible, so lineage hierarchies like stem-cell-centered trajectories can be read directly off the embedding.

## When to use

Use when you have scRNA-seq or protein expression profiles with prior RaceID2/cluster class labels and need a 2D visualization that exposes differentiation pathways and lineage relations (e.g. stem cell clusters, branch points) beyond what unsupervised t-SNE/PCA/MDS shows.

## When NOT to use

- You have no prior class/cluster assignment for the cells — GraphDDP starts from a user-defined cluster assignment and is not an unsupervised embedding
- You need pseudo-time ordering or quantitative trajectory scores — pseudotime-based methods like TSCAN or Monocle2 address that; GraphDDP is a visualization of hierarchy and pathways
- Classes are smaller than the min_threshold of 5 or larger than the max_threshold of 300 instances per class (tool defaults may drop classes)

## Inputs

- CSV expression/protein data matrix (one row per cell/instance), e.g. 462-cell intestinal epithelial scRNA-seq matrix
- CSV target file with discrete class labels per instance (e.g. RaceID2 cluster assignments)
- Optional parameters: -c class_confidence, -k, -d, -z, -l, --min_threshold (default 5), --max_threshold (default 300)

## Outputs

- Timestamped output directory containing 2D_coords.txt (2D coordinates, one line per instance)
- target.txt (target identifier, one line per instance)
- Embedding images img_1_clean.pdf through img_5_hull_link.pdf (clean layout, knn/k-shift edges, class convex hulls and first shift-edges)
- Execution log file

## How to apply

Install GraphEmbed (pip) with the non-pip dependency pygraphviz (conda install pygraphviz), or the python2 variant via conda from bioconda. Prepare the expression matrix (e.g. 462 intestinal epithelial cells from Grün et al.) as input CSV and the RaceID2 cluster assignments as target CSV, then run ./graph_embed.py with -i/-t. Enable the filter steps --normalization, --correlation_transformation (converts the matrix to a correlation-coefficient matrix) and --feature_selection, and set -c class separation strength (0.5-1 for clean well-separated clusters, 5-30 for noisy data), -k same-class knn links (default 3), -d denser different-class links (default 1), -z search horizon (default 5) and -l outlier mutual-neighbor threshold (default 3). The tool builds knn edges (same label, desired length divided by 1+class_confidence) and k_shift edges (toward denser instances of different label, computed from cosine-similarity density, dropped when an endpoint is an outlier with no mutual k neighbors), then lays out the graph with the Kamada-Kawai force algorithm. Inspect the produced img_1_clean.pdf through img_5_hull_link.pdf: on the intestinal epithelial data the correct result reproduces the reported Fig. 2c topology — stem cell cluster 7 centered, cluster 1 as common progenitor of Paneth and goblet cells, and distinct Cck+/Sst+ versus Gip+/Tac1+ enteroendocrine branches.

## Related tools

- **GraphDDP** (The supervised graph-embedding visualization approach this skill implements, integrating cluster-based and trajectory-based viewpoints) — https://github.com/fabriziocosta/GraphEmbed
- **GraphEmbed** (PyPI/conda-installable implementation of graph_embed.py that computes the 2D embedding from input and target CSVs) — https://github.com/fabriziocosta/GraphEmbed
- **pygraphviz** (Non-pip graphviz dependency to install via conda before running GraphEmbed)
- **TSCAN** (Comparison tool: cluster-based MST pseudotime method whose discretized cluster view GraphDDP complements; not executed by this skill) — https://github.com/zji90/TSCAN

## Examples

```
./graph_embed.py -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 -l 0 --random_state 2 --normalization --correlation_transformation --feature_selection
```

## Evaluation signals

- 2D_coords.txt and target.txt each contain exactly one line per input instance
- On the intestinal epithelial dataset, img_1_clean.pdf shows stem cell cluster 7 in the center with lineage clusters connected by differentiation trajectories (expert visual review vs. reported Fig. 2c)
- img_5_hull_link.pdf shows cluster 1 linked as common progenitor of Paneth and goblet clusters and separate Cck+/Sst+ vs Gip+/Tac1+ enteroendocrine branches
- With 0.5-1 class_confidence, same-class cells form compact, well-separated hulls; with 5-30, forced separation is visible on noisy data
- Output directory name carries the expected year-month-day-hour-min-sec timestamp suffix (unless --do_not_add_timestamp)

## Limitations

- No quantitative performance metrics are reported supporting superiority over t-SNE, PCA or MDS; the claim rests on visual identification of trajectories
- No software versions or parameter settings for compared tools (TSCAN, Monocle2, SPRING) and no dataset accessions for the intestinal epithelial or myeloid progenitor data are given
- No explicit limitations or failure cases of the method are discussed in the paper; validation of the layout is expert visual review rather than automated
- Classical dimensionality-reduction biases remain relevant: PCA may explain little variance, MDS can distort local arrangement, t-SNE depends on perplexity — GraphDDP addresses this only by injecting class information

## Evidence

- [abstract] GraphDDP integrates cluster-based and trajectory-based viewpoints of single-cell data in an intuitive visualization: "uses a force-based graph layout approach on two types of carefully constructed edges: one emphasizing cluster membership, the other, based on density gradients, emphasizing differentiation"
- [readme] Edge construction rationale: "A knn edge is an edge to the k-th nearest instance that has the same label. A k_shift edge is an edge to the k-th nearest instance that is denser and has a different label."
- [readme] Parameter guidance: "Values of 0.5-1 are suitable for clean data where clusters are naturally well separated, values of 5-30 are suitable for noisy data"
- [methods] Filter steps: "--correlation_transformation      Convert data matrix to corr coeff matrix."
- [readme] Layout algorithm: "the embedding is computed as the 2D coordinates of the corresponding graph embedding using the force layout algorithm from Tomihisa Kamada, and Satoru Kawai."
- [abstract] Dataset finding: "We show on intestinal epithelial cells and myeloid progenitor data"
