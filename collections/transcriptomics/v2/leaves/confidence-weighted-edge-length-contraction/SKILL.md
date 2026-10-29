---
name: confidence-weighted-edge-length-contraction
description: 'Use when a supervised 2D embedding must balance cluster separation against
  trajectory structure: discrete class labels exist per instance and the strength
  of trust in those labels (clean vs noisy single-cell data) needs an explicit, tunable
  bias.'
when_to_use_negative:
- 'Instances lack a discrete class label: GraphEmbed expects ''A discrete label for
  each instance'', so the same-class contraction cannot be applied.'
- The goal is pseudotime ordering on a cluster-level minimum spanning tree rather
  than an informed 2D layout - TSCAN's cluster-then-MST pipeline is the appropriate
  tool.
- A purely unsupervised dimensionality-reduction baseline (PCA, t-SNE, MDS) is required,
  since injecting class-confidence bias violates the unsupervised setup.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: GraphEmbed
  role: Implements graph_embed.py whose -c class_confidence parameter divides same-class
    desired distances by 1+C inside the Kamada-Kawai force layout; installable via
    PyPI.
  repo: github.com/fabriziocosta/GraphEmbed
- name: graph_embed
  role: Python2 conda distribution (bioconda) of the same embedding script used to
    execute the C-sweep layouts.
- name: pygraphviz
  role: Non-pip dependency (conda install pygraphviz) needed by GraphEmbed to render
    the embedding images such as img_1_clean.pdf.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:5760ad7ef83ebbe13ae19726e59bbb50139872f1ff41cb97a2baa44553ff7f77
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphDDP
  - GraphEmbed
  - graph_embed
  - pygraphviz
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# confidence-weighted-edge-length-contraction

## Summary

Tunes the supervised bias of the GraphDDP/GraphEmbed Kamada-Kawai force layout: desired distances on edges whose endpoints share a class label are contracted to d/(1+C), where C is the class-confidence parameter (-c, default 1.0). This trades off cluster-membership information against density-gradient trajectory information, with C=0.5-1 for clean data and C=5-30 for noisy data.

## When to use

Use when a supervised 2D embedding must balance cluster separation against trajectory structure: discrete class labels exist per instance and the strength of trust in those labels (clean vs noisy single-cell data) needs an explicit, tunable bias.

## When NOT to use

- Instances lack a discrete class label: GraphEmbed expects 'A discrete label for each instance', so the same-class contraction cannot be applied.
- The goal is pseudotime ordering on a cluster-level minimum spanning tree rather than an informed 2D layout - TSCAN's cluster-then-MST pipeline is the appropriate tool.
- A purely unsupervised dimensionality-reduction baseline (PCA, t-SNE, MDS) is required, since injecting class-confidence bias violates the unsupervised setup.

## Inputs

- Instance data matrix in CSV format (e.g. example/prot_expression.csv), optionally preprocessed with --normalization and/or --correlation_transformation
- Discrete class label file in CSV format, one label per instance (example/target.csv)
- Confidence bias value(s) C for the -c/--class_confidence option (default 1.0)

## Outputs

- 2D_coords.txt - one line of 2D embedding coordinates per instance
- target.txt - target identifier, one line per instance
- img_1_clean.pdf (plus img_2_edges.pdf, img_3_edge.pdf, img_4_hull.pdf, img_5_hull_link.pdf) - embedding images in the timestamped -o output directory
- log - program execution log file

## How to apply

Build the neighbor graph from the CSV data matrix (e.g. example/prot_expression.csv) and target CSV (example/target.csv): knn edges link each instance to its k=3 nearest same-label neighbors, and k_shift edges (d=1) link to denser different-class instances within a z=5 horizon, where density is the sum of pairwise cosine similarities between an instance and all others. Delete any k-shift edge with an outlier endpoint, defined as an instance with fewer than l=3 mutual nearest neighbors. Compute all-pairs desired distances on this graph via Dijkstra shortest paths (desired edge length is the Euclidean distance between endpoints), then apply the contraction rule: for every edge whose endpoints share the same label, divide the desired distance by 1+C, where C is the class-confidence (-c, default 1.0). Minimize the resulting Kamada-Kawai spring energy to obtain 2D coordinates, holding --random_state=1 fixed. Sweep C across the documented regimes - 0.5-1 for clean data with naturally well-separated clusters, 5-30 for noisy data requiring forced separation - so higher C compacts same-class layouts while density-gradient edges still trace differentiation pathways.

## Related tools

- **GraphEmbed** (Implements graph_embed.py whose -c class_confidence parameter divides same-class desired distances by 1+C inside the Kamada-Kawai force layout; installable via PyPI.) — github.com/fabriziocosta/GraphEmbed
- **graph_embed** (Python2 conda distribution (bioconda) of the same embedding script used to execute the C-sweep layouts.)
- **pygraphviz** (Non-pip dependency (conda install pygraphviz) needed by GraphEmbed to render the embedding images such as img_1_clean.pdf.)

## Examples

```
./graph_embed -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 -l 0 --random_state 2
```

## Evaluation signals

- Increasing C with fixed --random_state yields visibly more compact same-class layouts (shrinking class convex hulls in img_4_hull.pdf), matching 'more compact layouts for instances of the same class'.
- 2D_coords.txt contains exactly one line per input instance and target.txt has the same line count and order.
- The layout remains stable over a whole range of confidence values (Supplementary Fig. 1 behavior) instead of collapsing or reordering between adjacent C values.
- img_2_edges.pdf shows red knn edges and blue k-shift edges, with k-shift edges absent for outlier endpoints (fewer than l mutual neighbors).
- Output directory name is suffixed with a year-month-day-hour-min-sec timestamp and contains 2D_coords.txt, target.txt, img_1_clean.pdf and the log file.

## Limitations

- No quantitative performance metrics are reported supporting GraphDDP over t-SNE, PCA, or MDS - the claimed superiority is qualitative.
- No dataset accessions for the intestinal epithelial and myeloid progenitor data and no software versions/parameter settings for compared tools (TSCAN, Monocle2, SPRING) are given.
- Appropriate C is data-dependent (0.5-1 clean vs 5-30 noisy); an unsuitable value under- or over-compacts the same-class layout, and k-shift edges to outliers are deleted, potentially dropping differentiation signal for sparse cell types.

## Evidence

- [readme] If the endpoints of an edge have the same label then the desired distance is divided by 1 + class_confidence.: "If the endpoints of an edge have the same label then the desired distance is divided by 1 + class_confidence."
- [readme] Values of 0.5-1 are suitable for clean data where clusters are naturally well separated, values of 5-30 are suitable for noisy data: "Values of 0.5-1 are suitable for clean data where clusters are naturally well separated, values of 5-30 are suitable for noisy data"
- [readme] Values higher than 0 indicate a stronger belief and will result in more compact layouts for instances of the same class.: "Values higher than 0 indicate a stronger belief and will result in more compact layouts for instances of the same class."
- [readme] A k-shift edge is deleted if at least one of the endpoints is an outlier.: "A k-shift edge is deleted if at least one of the endpoints is an outlier."
- [methods] To set a desired separation strength use -c, to specify the number of neighbors use -k: "To set a desired separation strength use -c, to specify the number of neighbors use -k"
- [abstract] uses a force-based graph layout approach on two types of carefully constructed edges: one emphasizing cluster membership, the other, based on density gradients, emphasizing differentiation: "uses a force-based graph layout approach on two types of carefully constructed edges: one emphasizing cluster membership, the other, based on density gradients, emphasizing differentiation"
- [supplementary] the layout remains stable over a whole range of possible confidence values (see Supplementary Fig. 1): [evidence span withheld — anchor status: not_found]
