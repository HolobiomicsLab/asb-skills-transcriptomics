---
name: force-directed-graph-layout-implementation
description: Use when a supervised 2D embedding must preserve both graph connectivity
  (knn and k_shift edges) and prior class labels, e.g. laying out a nearest-neighbor
  graph with Kamada-Kawai spring-energy minimization and class-confidence-contracted
  same-class distances.
when_to_use_negative:
- 'Instances carry no discrete class label: the same-class distance contraction d/(1
  + class_confidence) and the knn edges require supervised labels (''A discrete label
  for each instance is expected'').'
- The analysis goal is a pseudotime or trajectory ordering rather than a 2D visualization
  — cluster-MST tools like TSCAN address pseudotime reconstruction instead.
- A 3D or higher-dimensional embedding is needed — this implementation produces 2D
  coordinates only.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: GraphEmbed
  role: Reference implementation that builds the knn/k_shift graph and computes the
    Kamada-Kawai 2D embedding with class-confidence contraction; installable via PyPI
    (pip install graphembed).
  repo: github.com/fabriziocosta/GraphEmbed
- name: graph_embed
  role: Python2 CLI variant of the same layout pipeline distributed via bioconda (conda
    install graph_embed -c bioconda); executes the -c/-k/-d/-z/-l parameterized embedding
    end to end.
  repo: github.com/fabriziocosta/GraphEmbed
- name: pygraphviz
  role: Non-pip graphviz dependency required by GraphEmbed for graph handling/rendering
    during layout output (conda install pygraphviz).
- name: GraphDDP
  role: Article's method that applies the confidence-contracted force layout to single-cell
    data to visualize differentiation pathways; relies on the GraphEmbed implementation.
  repo: github.com/fabriziocosta/GraphEmbed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:effb224bac98c0bb73d5186d924a2767ffd84b534cc1c6073b528e83046b1fb6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphDDP
  - GraphEmbed
  - graph_embed
  - pygraphviz
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# force-directed-graph-layout-implementation

## Summary

Implementation of the Kamada-Kawai force-directed graph layout used by GraphDDP/GraphEmbed, in which the desired distance between same-class instances is contracted by a class-confidence factor (d/(1 + class_confidence)). This merges the cluster-membership and density-gradient trajectory views into a single supervised 2D embedding.

## When to use

Use when a supervised 2D embedding must preserve both graph connectivity (knn and k_shift edges) and prior class labels, e.g. laying out a nearest-neighbor graph with Kamada-Kawai spring-energy minimization and class-confidence-contracted same-class distances.

## When NOT to use

- Instances carry no discrete class label: the same-class distance contraction d/(1 + class_confidence) and the knn edges require supervised labels ('A discrete label for each instance is expected').
- The analysis goal is a pseudotime or trajectory ordering rather than a 2D visualization — cluster-MST tools like TSCAN address pseudotime reconstruction instead.
- A 3D or higher-dimensional embedding is needed — this implementation produces 2D coordinates only.

## Inputs

- Feature data matrix in CSV format (e.g., example/prot_expression.csv), passed via -i
- Discrete class label per instance in CSV format (e.g., example/target.csv), passed via -t
- Layout parameters: -c class_confidence (default 1.0), -k same-class neighbor links (default 3), -d denser cross-class links (default 1), -z neighbor-search horizon (default 5), -l mutual-neighbor outlier threshold (default 3), --random_state seed (default 1)

## Outputs

- 2D_coords.txt — the 2D embedding coordinates, one line per instance
- target.txt — the target identifier, one line per instance
- img_1_clean.pdf — image of the 2D embedding (img_2_edges.pdf, img_3_edge.pdf, img_4_hull.pdf, img_5_hull_link.pdf give edge- and hull-annotated views)
- log — program execution log, all files written to an output directory (-o) whose name is suffixed with a year-month-day-hour-min-sec timestamp

## How to apply

Build a graph whose nodes are instances with two edge types: 'knn' edges to the -k (default 3) nearest same-label instances and 'k_shift' edges to the -d (default 1) nearest denser different-label instances within a -z (default 5) neighbor horizon, where density is the sum of pairwise cosine similarities between an instance and all other instances. Delete k_shift edges having an outlier endpoint, outliers being instances with fewer than -l (default 3) mutual nearest neighbors. Compute all-pairs desired distances as Dijkstra shortest paths over this graph, using the euclidean distance between endpoints as edge length and dividing it by (1 + class_confidence) when both endpoints share a label (-c, default 1.0). Minimize the Kamada-Kawai (1989) spring energy to obtain the 2D coordinates, fixing --random_state (default 1) for reproducibility. Sweep -c across the recommended regimes — 0.5-1 for clean, well-separated clusters and 5-30 for noisy data — and verify that higher C yields visibly more compact same-class layouts while the overall layout remains stable over the whole confidence range.

## Related tools

- **GraphEmbed** (Reference implementation that builds the knn/k_shift graph and computes the Kamada-Kawai 2D embedding with class-confidence contraction; installable via PyPI (pip install graphembed).) — github.com/fabriziocosta/GraphEmbed
- **graph_embed** (Python2 CLI variant of the same layout pipeline distributed via bioconda (conda install graph_embed -c bioconda); executes the -c/-k/-d/-z/-l parameterized embedding end to end.) — github.com/fabriziocosta/GraphEmbed
- **pygraphviz** (Non-pip graphviz dependency required by GraphEmbed for graph handling/rendering during layout output (conda install pygraphviz).)
- **GraphDDP** (Article's method that applies the confidence-contracted force layout to single-cell data to visualize differentiation pathways; relies on the GraphEmbed implementation.) — github.com/fabriziocosta/GraphEmbed

## Examples

```
./graph_embed -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 -l 0 --random_state 2 --normalization --correlation_transformation --feature_selection
```

## Evaluation signals

- 2D_coords.txt contains exactly one line per input instance with two numeric coordinates, target.txt has one label per instance, and img_1_clean.pdf renders the embedding in the timestamped -o directory.
- Increasing -c produces visibly more compact same-class clusters, consistent with the documented contraction of same-class distances to d/(1 + class_confidence).
- The layout remains stable across a sweep of confidence values (Supplementary Fig. 1): global trajectory structure does not reorganize when moving between the C=0.5-1 and C=5-30 regimes.
- img_2_edges.pdf shows red knn edges (-k same-class links per node) and blue k-shift edges (-d cross-class links within the -z horizon); k-shift edges are absent at outliers lacking -l mutual neighbors.
- Repeated runs with the same --random_state reproduce identical layouts, and the output directory name carries the year-month-day-hour-min-sec timestamp suffix.

## Limitations

- Requires a user-defined cluster assignment (discrete label per instance); without reliable class information the confidence term cannot meaningfully trade off cluster membership against trajectory edges.
- k_shift (trajectory) edges are deleted if either endpoint is an outlier (an instance with no mutual k neighbors), so sparse or isolated cells lose differentiation edges.
- Large confidence values (5-30) are only suitable for noisy data where a strong separation must be forced; they contract same-class distances to d/(1+C) and can distort within-class geometry.
- No quantitative performance metrics were reported against t-SNE, PCA or MDS — superiority claims rest on qualitative visualization of intestinal epithelial and myeloid progenitor data.

## Evidence

- [abstract] uses a force-based graph layout approach on two types of carefully constructed edges: one emphasizing cluster membership, the other, based on density gradients, emphasizing differentiation: "uses a force-based graph layout approach on two types of carefully constructed edges: one emphasizing cluster membership, the other, based on density gradients, emphasizing differentiation"
- [readme] If the endpoints of an edge have the same label then the desired distance is divided by 1 + class_confidence.: "If the endpoints of an edge have the same label then the desired distance is divided by 1 + class_confidence."
- [readme] Finally the embedding is computed as the 2D coordinates of the corresponding graph embedding using the force layout algorithm from Tomihisa Kamada, and Satoru Kawai.: "Finally the embedding is computed as the 2D coordinates of the corresponding graph embedding using the force layout algorithm from Tomihisa Kamada, and Satoru Kawai."
- [readme] Values higher than 0 indicate a stronger belief and will result in more compact layouts for instances of the same class.: "Values higher than 0 indicate a stronger belief and will result in more compact layouts for instances of the same class."
- [readme] A k-shift edge is deleted if at least one of the endpoints is an outlier.: "A k-shift edge is deleted if at least one of the endpoints is an outlier."
- [methods] To set a desired separation strength use -c, to specify the number of neighbors use -k: "To set a desired separation strength use -c, to specify the number of neighbors use -k"
- [supplementary] the layout remains stable over a whole range of possible confidence values: [evidence span withheld — anchor status: not_found]
