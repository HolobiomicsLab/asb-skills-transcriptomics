---
name: force-directed-2d-embedding-visualization
description: Use when a single-cell or labeled expression matrix (CSV) already has
  discrete cluster assignments and you need a 2D visualization that integrates cluster
  membership with trajectory-like transitions between clusters (e.g. differentiation
  pathways).
when_to_use_negative:
- Instances have no discrete class/cluster labels available — GraphDDP starts from
  a user-defined cluster assignment, so unsupervised data cannot drive the supervised
  knn edges
- The goal is quantitative pseudotime ordering or gene-expression dynamics along a
  tree rather than 2D visualization — pseudotime tools such as TSCAN (cluster-based
  MST ordering) are the appropriate choice
- Any class has fewer than 5 or more than 300 instances relative to the --min_threshold/--max_threshold
  defaults, e.g. very small or very large single clusters that violate the instance-per-class
  bounds
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: GraphEmbed
  role: PyPI-installable implementation that builds the knn/k_shift graph and computes
    the Kamada–Kawai 2D force layout embedding
  repo: github.com/fabriziocosta/GraphEmbed
- name: graph_embed
  role: Python2 conda/bioconda variant of the graph-embedding executable invoked as
    ./graph_embed.py with -i/-t and edge-strength options
  repo: bioconda
- name: pygraphviz
  role: Non-pip dependency (installed via conda) required by GraphEmbed
- name: TSCAN
  role: Alternative cluster-based MST pseudotime tool; not used for the force-directed
    2D layout itself but contrasts with it in the article's discussion
  repo: zji90/TSCAN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:458b1e91d96ad230c645b18bbdad03c7b378753a6466cb8c653e65a5171732b3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphDDP
  - GraphEmbed
  - TSCAN
  - graph_embed
  - pygraphviz
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# force-directed-2d-embedding-visualization

## Summary

Compute a supervised 2D embedding of a data matrix using GraphDDP/GraphEmbed: a force-based (Kamada–Kawai) graph layout over 'knn' edges that compress same-class clusters and density-gradient 'k_shift' edges that connect clusters along differentiation trajectories.

## When to use

Use when a single-cell or labeled expression matrix (CSV) already has discrete cluster assignments and you need a 2D visualization that integrates cluster membership with trajectory-like transitions between clusters (e.g. differentiation pathways).

## When NOT to use

- Instances have no discrete class/cluster labels available — GraphDDP starts from a user-defined cluster assignment, so unsupervised data cannot drive the supervised knn edges
- The goal is quantitative pseudotime ordering or gene-expression dynamics along a tree rather than 2D visualization — pseudotime tools such as TSCAN (cluster-based MST ordering) are the appropriate choice
- Any class has fewer than 5 or more than 300 instances relative to the --min_threshold/--max_threshold defaults, e.g. very small or very large single clusters that violate the instance-per-class bounds

## Inputs

- Input data matrix in CSV format (e.g. scRNA-seq expression matrix of 462 intestinal epithelial cells, rows = instances)
- Target classes file in CSV format with one discrete cluster label per instance (e.g. RaceID2 cluster assignments)

## Outputs

- 2D_coords.txt — 2D coordinates, one line per instance
- target.txt — target identifier, one line per instance
- img_1_clean.pdf — clean 2D embedding image
- img_2_edges.pdf — embedding with red knn edges and blue k-shift edges
- img_3_edge.pdf — embedding with blue first shift-edges
- img_4_hull.pdf — embedding with class convex hulls and edge thickness proportional to cross-class shift-edge counts
- img_5_hull_link.pdf — embedding with class convex hulls and first shift-edges
- log — execution log; all files written to a timestamped output directory

## How to apply

Start from a user-defined cluster assignment (target CSV with one discrete label per instance) and an input data CSV; optionally run --normalization, --correlation_transformation (to a Pearson correlation-coefficient matrix) and --feature_selection to emphasize discriminative features. Build a graph where each 'knn' edge links to the k-th nearest same-label instance (desired distance divided by 1 + class_confidence) and each 'k_shift' edge links to the k-th nearest denser different-label instance, with density defined as the sum of pairwise cosine similarities to all other instances; delete k_shift edges touching outliers (instances with no mutual k neighbors, threshold -l, default 3). Tune -c (class separation strength: 0.5–1 for clean well-separated clusters, 5–30 for noisy data), -k (same-class neighbors, default 3), -d (denser cross-class links, default 1) and -z (search horizon, default 5). Compute the 2D embedding with the Kamada–Kawai force layout and inspect the five output images (img_1_clean through img_5_hull_link) to read cluster compaction and inter-class shift-edge topology as trajectories.

## Related tools

- **GraphEmbed** (PyPI-installable implementation that builds the knn/k_shift graph and computes the Kamada–Kawai 2D force layout embedding) — github.com/fabriziocosta/GraphEmbed
- **graph_embed** (Python2 conda/bioconda variant of the graph-embedding executable invoked as ./graph_embed.py with -i/-t and edge-strength options) — bioconda
- **pygraphviz** (Non-pip dependency (installed via conda) required by GraphEmbed)
- **TSCAN** (Alternative cluster-based MST pseudotime tool; not used for the force-directed 2D layout itself but contrasts with it in the article's discussion) — zji90/TSCAN

## Examples

```
./graph_embed.py -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 -l 0 --random_state 2 --normalization --correlation_transformation --feature_selection
```

## Evaluation signals

- 2D_coords.txt contains exactly one coordinate line per input instance and target.txt one label per instance, both in instance order
- With -c > 0, same-label instances form visibly compact clusters in img_1_clean.pdf because knn-edge desired distances are divided by (1 + class_confidence)
- k_shift edges only connect instances to denser different-label neighbors within the -z horizon, and no shift edge touches an outlier lacking -l mutual nearest neighbors
- Output directory is suffixed with a year-month-day-hour-min-sec timestamp and contains all five img_*.pdf images plus the log file
- Biological validation: on the intestinal epithelial dataset the layout should place stem cell cluster 7 centrally and connect lineage clusters (cluster 1 as common Paneth/goblet progenitor; separated Cck+/Sst+ vs Gip+/Tac1+ enteroendocrine branches), matching the reported Fig. 2c topology

## Limitations

- No quantitative performance metrics are reported supporting superiority over t-SNE, PCA or MDS; validation is expert visual review of topology
- Class separation quality depends on -c tuning: 0.5–1 only suits clean well-separated clusters, noisy data requires forcing 5–30, and random_state changes the layout
- The python2 conda version is limited to --min_threshold 5 / --max_threshold 300 instances per class by default; k_shift edges are deleted for outliers, which can drop trajectory links for rare (e.g. stem) populations

## Evidence

- [abstract] Force layout on knn and k_shift edges: "uses a force-based graph layout approach on two types of carefully constructed edges: one emphasizing cluster membership, the other, based on density gradients, emphasizing differentiation"
- [readme] Kamada–Kawai embedding definition: "Finally the embedding is computed as the 2D coordinates of the corresponding graph embedding using the force layout algorithm from"
- [readme] knn edge semantics: "A knn edge is an edge to the k-th nearest instance that has the same label."
- [readme] Density definition for shift edges: "Density is defined as the sum of the pairwise cosine similarity between an instance and all the other instances."
- [readme] Outlier rule: "Outlier nodes are defined as those instances that have no mutual k neighbors."
- [readme] Class confidence tuning guidance: "Values of 0.5-1 are suitable for clean data where clusters are naturally well separated, values of 5-30 are suitable for noisy data"
