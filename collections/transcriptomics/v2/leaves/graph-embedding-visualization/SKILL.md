---
name: graph-embedding-visualization
description: Use when you have a data matrix plus a user-defined cluster/class assignment
  (CSV) and need a supervised 2D layout that simultaneously reveals cluster structure
  and density-gradient differentiation trajectories, as in the intestinal epithelial
  and myeloid progenitor datasets.
when_to_use_negative:
- No prior cluster/class assignment exists for the instances — GraphDDP is supervised
  and requires a discrete label per instance
- The goal is pseudotime ordering or trajectory inference alone (e.g. pseudo-time
  reconstruction) rather than an informed 2D visualization of cluster membership and
  differentiation pathways
- The dataset's clusters are naturally well separated and unsupervised methods like
  t-SNE, PCA, or MDS already suffice — reserve -c values of 5-30 and forced separation
  for noisy data
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: GraphEmbed
  role: Python package implementing the knn/k_shift graph builder and Kamada-Kawai
    force layout used for the 2D embedding
  repo: github.com/fabriziocosta/GraphEmbed
- name: graph_embed
  role: Python2/conda command-line entry point (graph_embed.py) executed with -i/-t
    inputs and edge parameters -k, -d, -z, -l, -c
  repo: github.com/fabriziocosta/GraphEmbed
- name: pygraphviz
  role: Non-pip dependency required for graph handling/installation
- name: Monocle2
  role: Comparison pseudotime tool (DDRTree layout, min_expr=20) used to contrast
    its lineage resolution against the GraphDDP embedding
- name: TSCAN
  role: Cluster-based MST pseudotime method cited as an alternative approach for comparison
  repo: zji90/TSCAN
provenance:
  source_task_ids:
  - task_005
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:846ebcd06721a4edb93959dd455ef29404b26652268103979613c390f6e54919
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphDDP
  - GraphEmbed
  - Monocle2
  - TSCAN
  - graph_embed
  - pygraphviz
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# graph-embedding-visualization

## Summary

Compute a 2D embedding of single-cell data via GraphDDP/GraphEmbed, which builds a graph with 'knn' edges (same-label nearest neighbors) and 'k_shift' edges (denser, different-label neighbors) and lays it out with the Kamada-Kawai force algorithm to integrate cluster membership and differentiation pathways in one visualization.

## When to use

Use when you have a data matrix plus a user-defined cluster/class assignment (CSV) and need a supervised 2D layout that simultaneously reveals cluster structure and density-gradient differentiation trajectories, as in the intestinal epithelial and myeloid progenitor datasets.

## When NOT to use

- No prior cluster/class assignment exists for the instances — GraphDDP is supervised and requires a discrete label per instance
- The goal is pseudotime ordering or trajectory inference alone (e.g. pseudo-time reconstruction) rather than an informed 2D visualization of cluster membership and differentiation pathways
- The dataset's clusters are naturally well separated and unsupervised methods like t-SNE, PCA, or MDS already suffice — reserve -c values of 5-30 and forced separation for noisy data

## Inputs

- Input data matrix in CSV format (e.g. example/prot_expression.csv)
- Class/cluster assignment target file in CSV format (e.g. example/target.csv)
- Optional parameters: -c class confidence, -k same-class neighbor links, -d denser cross-class links, -z density-gradient search horizon, -l mutual-neighbor outlier count

## Outputs

- 2D_coords.txt — 2D coordinates, one line per instance
- target.txt — target identifier, one line per instance
- img_1_clean.pdf through img_5_hull_link.pdf — 2D embedding images showing knn edges (red), k-shift edges (blue), and class convex hulls
- log — program execution log file
- Timestamped output directory set via -o

## How to apply

Load the expression matrix (-i CSV) and class assignments (-t CSV), optionally applying --normalization, --correlation_transformation, or --feature_selection and bounding class sizes with --min_threshold=5/--max_threshold=300. The builder adds a 'knn' edge from each instance to its k-th nearest same-label instance (-k, default 3), excluding outliers, and a 'k_shift' edge (quick-shift style) to the d nearest denser instances with a different label (-d, default 1) searched within the horizon of the z nearest neighbors (-z, default 5); density is the sum of pairwise cosine similarities to all other instances. Outliers are instances lacking the required number of mutual nearest neighbors (-l, default 3) and any k-shift edge touching an outlier is deleted. Edge desired lengths are Euclidean distances between instances, divided by 1 + class_confidence when endpoints share a label (-c 0.5-1 for clean clusters, 5-30 for noisy data with strong supervised belief). The final 2D coordinates are computed with the force layout of Kamada and Kawai, and outputs (2D_coords.txt, target.txt, img_1_clean.pdf through img_5_hull_link.pdf, log) go to a timestamped directory set with -o.

## Related tools

- **GraphEmbed** (Python package implementing the knn/k_shift graph builder and Kamada-Kawai force layout used for the 2D embedding) — github.com/fabriziocosta/GraphEmbed
- **graph_embed** (Python2/conda command-line entry point (graph_embed.py) executed with -i/-t inputs and edge parameters -k, -d, -z, -l, -c) — github.com/fabriziocosta/GraphEmbed
- **pygraphviz** (Non-pip dependency required for graph handling/installation)
- **Monocle2** (Comparison pseudotime tool (DDRTree layout, min_expr=20) used to contrast its lineage resolution against the GraphDDP embedding)
- **TSCAN** (Cluster-based MST pseudotime method cited as an alternative approach for comparison) — zji90/TSCAN

## Examples

```
./graph_embed.py -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 --normalization --correlation_transformation --feature_selection --random_state 2
```

## Evaluation signals

- 2D_coords.txt contains exactly one coordinate line per input instance and target.txt one target line per instance
- img_2_edges.pdf is a valid PDF that renders knn edges in red and k-shift edges in blue
- The class convex-hull layout (img_4_hull.pdf / img_5_hull_link.pdf) recovers the expected meta-cluster adjacency, e.g. the erythrocyte C7-C1 ordering in the Paul et al. myeloid progenitor data
- Ternary-plot confusion scores computed from shift-edges match reported values (e.g. 0.06 for the C5-C6-C7 erythrocyte transition, 0.72 for C2-C3-C4)
- With a fixed --random_state, reruns produce the same timestamped output contents for identical inputs and parameters

## Limitations

- No quantitative performance metrics are reported to support claimed superiority over t-SNE, PCA, or MDS; comparisons are visual/qualitative
- No software versions or parameter settings are given for the compared tools (TSCAN, Monocle2, SPRING)
- No dataset accessions are cited for the intestinal epithelial and myeloid progenitor datasets, and no explicit limitations or failure cases are discussed in the paper
- k-shift edges are deleted if either endpoint is an outlier (no mutual k neighbors), so rare/isolated cells may lose trajectory links

## Evidence

- [abstract] Two edge types encode cluster membership and differentiation: "uses a force-based graph layout approach on two types of carefully constructed edges: one emphasizing cluster membership, the other, based on density gradients, emphasizing differentiation"
- [readme] knn and k_shift edge definitions from README: "A knn edge is an edge to the k-th nearest instance that has the same label. A k_shift edge is an edge to the k-th nearest instance that is denser and has a different label."
- [readme] Density definition and confidence-scaled edge length: "Density is defined as the sum of the pairwise cosine similarity between an instance and all the other instances."
- [readme] Class confidence guidance for clean vs noisy data: "Values of 0.5-1 are suitable for clean data where clusters are naturally well separated, values of 5-30 are suitable for noisy data where it is necessary to force a strong separation"
- [readme] Kamada-Kawai force layout reference: "the embedding is computed as the 2D coordinates of the corresponding graph embedding using the force layout algorithm from Tomihisa Kamada, and Satoru Kawai."
- [methods] Outputs listing: "The option -o allows to set a directory where the produced files are placed. The directory name is suffixed with a time stamp"
