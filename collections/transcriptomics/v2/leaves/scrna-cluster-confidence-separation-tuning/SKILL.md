---
name: scrna-cluster-confidence-separation-tuning
description: Use when producing a 2D graph embedding of scRNA-seq cells with known
  cluster labels and the layout is either too diffuse for same-class cells or overly
  collapsed, requiring adjustment of -c, -k, -d, -z, -l.
when_to_use_negative:
- No prior cluster/class labels exist for the cells — the method requires a discrete
  label per instance and cannot run from raw expression alone
- The goal is a purely unsupervised 2D visualization without integrating a prior clustering
  phase; classical t-SNE, PCA or MDS would be the appropriate choice
- The data are not single-cell profiles with a continuous differentiation structure
  where forcing compact same-class clusters would obscure trajectory topology
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: GraphEmbed
  role: Computes the 2D supervised force-based (Kamada-Kawai) graph embedding whose
    separation strength and neighbor parameters are tuned here; installed via PyPI
    or bioconda
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: graph_embed
  role: Python2 conda distribution of the same embedding program; CLI whose -c, -k,
    -d, -z, -l flags are the tuning knobs
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: pygraphviz
  role: Non-pip dependency required for the graph layout; installed via conda
- name: GraphDDP
  role: Downstream application of the tuned embedding to detect differentiation pathways
    using cluster-membership and density-gradient trajectory edges
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:52e93be7c3415351e488e5fe059b88c6a0ced2357733b586ddd0f285c7d20b8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fabriziocosta/GraphEmbed
  tools:
  - GraphDDP
  - GraphEmbed
  - graph_embed
  - pygraphviz
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# scrna-cluster-confidence-separation-tuning

## Summary

Tunes the class-separation confidence (-c) and neighbor-graph parameters (-k, -d, -z, -l) of GraphEmbed/GraphDDP so that prior cluster labels (e.g., RaceID2 clusters) are rendered as compact, well-separated groups in the 2D force-based embedding while preserving differentiation-trajectory edges.

## When to use

Use when producing a 2D graph embedding of scRNA-seq cells with known cluster labels and the layout is either too diffuse for same-class cells or overly collapsed, requiring adjustment of -c, -k, -d, -z, -l.

## When NOT to use

- No prior cluster/class labels exist for the cells — the method requires a discrete label per instance and cannot run from raw expression alone
- The goal is a purely unsupervised 2D visualization without integrating a prior clustering phase; classical t-SNE, PCA or MDS would be the appropriate choice
- The data are not single-cell profiles with a continuous differentiation structure where forcing compact same-class clusters would obscure trajectory topology

## Inputs

- Expression data matrix as CSV (-i), e.g. intestinal epithelial scRNA-seq matrix of 462 cells
- Cluster/class assignment file as CSV (-t), e.g. RaceID2 cluster labels, one discrete label per instance

## Outputs

- Timestamped output directory with 2D_coords.txt (one line of 2D coordinates per instance)
- target.txt (target identifier per instance)
- Embedding images img_1_clean.pdf through img_5_hull_link.pdf showing knn edges, k-shift edges, and class convex hulls
- Program execution log file

## How to apply

Run graph_embed.py on the expression matrix (-i CSV) with the cluster assignment file (-t CSV) and enable --normalization, --correlation_transformation, and --feature_selection so the graph is built on discriminative correlation features. Set the class separation strength via -c following the README guidance: values of 0.5-1 suit clean, naturally well-separated clusters (as in the 462-cell intestinal epithelial data with RaceID2 labels), while values of 5-30 suit noisy data needing forced separation. Adjust -k (same-class knn links, default 3), -d (denser different-class k-shift links, default 1), -z (search horizon, default 5), and -l (mutual-neighbor outlier threshold, default 3) to control edge construction; recall that same-label desired distances are divided by 1 + class_confidence, and k-shift edges are deleted when an endpoint is an outlier. Iterate over -c/-k values and inspect img_2_edges.pdf to verify red knn edges keep classes compact while blue k-shift edges trace density-gradient trajectories between clusters.

## Related tools

- **GraphEmbed** (Computes the 2D supervised force-based (Kamada-Kawai) graph embedding whose separation strength and neighbor parameters are tuned here; installed via PyPI or bioconda) — https://github.com/fabriziocosta/GraphEmbed
- **graph_embed** (Python2 conda distribution of the same embedding program; CLI whose -c, -k, -d, -z, -l flags are the tuning knobs) — https://github.com/fabriziocosta/GraphEmbed
- **pygraphviz** (Non-pip dependency required for the graph layout; installed via conda)
- **GraphDDP** (Downstream application of the tuned embedding to detect differentiation pathways using cluster-membership and density-gradient trajectory edges)

## Examples

```
./graph_embed.py -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 -l 0 --normalization --correlation_transformation --feature_selection
```

## Evaluation signals

- img_1_clean.pdf reproduces the expected topology (e.g., stem cell cluster 7 at the center of the intestinal epithelial layout with lineages on connected trajectories)
- In img_2_edges.pdf, red knn edges form compact intra-class groups while blue k-shift edges connect different classes along density gradients
- Same-class cells are drawn more compactly as -c increases, since same-label desired distances are divided by 1 + class_confidence; verify the -c regime (0.5-1 vs 5-30) matches data cleanliness
- Output directory contains all expected artifacts: 2D_coords.txt, target.txt, img_1_clean.pdf through img_5_hull_link.pdf, and the log, suffixed with a year-month-day-hour-min-sec timestamp
- k-shift edges are absent for outlier endpoints, confirming the -l mutual-neighbor outlier rule (default 3) is in effect

## Limitations

- No quantitative performance metrics are reported supporting superiority over t-SNE, PCA or MDS; assessment is expert visual review of the embedding images
- Choosing -c is data-dependent and qualitative: 0.5-1 for clean well-separated clusters, 5-30 for noisy data, with no automatic selection procedure
- Outliers (instances with no mutual k neighbors) lose their k-shift edges, so rare cells such as stem cells may lose trajectory links
- No dataset accessions or explicit failure cases are provided for the intestinal epithelial and myeloid progenitor benchmarks

## Evidence

- [readme] Values of 0.5-1 are suitable for clean data where clusters are naturally well separated, values of 5-30 are suitable for noisy data where it is necessary to force a strong separation in the 2D representation.: "Values of 0.5-1 are suitable for clean data where clusters are naturally well separated, values of 5-30 are suitable for noisy data where it is necessary to force a strong separation"
- [readme] If the endpoints of an edge have the same label then the desired distance is divided by 1 + class_confidence.: "If the endpoints of an edge have the same label then the desired distance is divided by 1 + class_confidence."
- [readme] A k-shift edge is deleted if at least one of the endpoints is an outlier. Outlier nodes are defined as those instances that have no mutual k neighbors.: "A k-shift edge is deleted if at least one of the endpoints is an outlier. Outlier nodes are defined as those instances that have no mutual k neighbors."
- [abstract] GraphDDP starts from a user-defined cluster assignment and then uses a force-based graph layout approach: "GraphDDP starts from a user-defined cluster assignment and then uses a force-based graph layout approach"
- [methods] To set a desired separation strength use -c, to specify the number of neighbors use -k: "To set a desired separation strength use -c, to specify the number of neighbors use -k"
