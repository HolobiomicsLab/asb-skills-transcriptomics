---
name: density-gradient-edge-construction
description: Use when you have single-cell expression data with a prior (e.g. RaceID3-derived)
  cluster assignment and need a 2D embedding whose topology encodes both class separation
  and density-gradient-based differentiation paths.
when_to_use_negative:
- When no prior cluster/class assignment exists — the knn and k_shift edges both depend
  on discrete labels for each instance
- When a quantitative pseudotime ordering (rather than a qualitative 2D hierarchy
  visualization) is the required output — pseudotime methods like TSCAN or Monocle2
  are the appropriate alternatives
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: GraphDDP
  role: Method whose force-based graph layout consumes the knn and density-gradient
    (k_shift) edges to visualize differentiation pathways
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: GraphEmbed
  role: Command-line implementation (graph_embed.py) that constructs the knn/k_shift
    edges and computes the 2D embedding; installable via PyPI or conda (graph_embed
    -c bioconda)
  repo: https://github.com/fabriziocosta/GraphEmbed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:bfebeb8ada6c4dd0014b22378ad1ce2b618300f6835bd4688bfcd424b76eea60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fabriziocosta/GraphEmbed
  tools:
  - GraphDDP
  - GraphEmbed
  - pygraphviz
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# density-gradient-edge-construction

## Summary

Construction of the two graph edge types used by GraphDDP/GraphEmbed: 'knn' edges linking each instance to its k-th nearest same-class neighbor and 'k_shift' edges linking toward denser instances of a different class, so a force-based (Kamada–Kawai) layout jointly reveals cluster membership and differentiation trajectories in single-cell data.

## When to use

Use when you have single-cell expression data with a prior (e.g. RaceID3-derived) cluster assignment and need a 2D embedding whose topology encodes both class separation and density-gradient-based differentiation paths.

## When NOT to use

- When no prior cluster/class assignment exists — the knn and k_shift edges both depend on discrete labels for each instance
- When a quantitative pseudotime ordering (rather than a qualitative 2D hierarchy visualization) is the required output — pseudotime methods like TSCAN or Monocle2 are the appropriate alternatives

## Inputs

- Expression data matrix in CSV format (-i), e.g. single-cell transcript counts per cell
- Target/classes CSV file (-t) with one discrete label per instance (e.g. RaceID3 cluster assignments)
- Edge-construction parameters: -k (same-class links, default 3), -d (denser-neighbor links, default 1), -z (density-gradient search horizon, default 5), -l (outlier mutual-neighbor count, default 3), -c (class separation strength)

## Outputs

- Graph of knn and k_shift edges embedding class membership and density-gradient trajectory structure
- 2D embedding coordinates (2D_coords.txt) and target identifiers (target.txt) in a timestamped output directory
- Edge visualization images img_2_edges.pdf–img_5_hull_link.pdf showing knn edges (red), k-shift edges (blue), and class convex hulls with cross-class shift-edge links

## How to apply

Start from an expression matrix CSV and a target CSV of class labels (e.g. RaceID3 cluster assignments for the Velten mouse hematopoietic progenitors). Build 'knn' edges to the k-th nearest instance with the same label (default k=3 via -k) and 'k_shift' edges to the k-th nearest instance that is denser and has a different label, where density is defined as the sum of pairwise cosine similarity between an instance and all others; limit the search horizon with -z (default 5) and set the number of denser-neighbor links with -d (default 1). Desired edge length is the Euclidean distance between instances, divided by 1 + class_confidence for same-label edges (tune -c: 0.5–1 for well-separated clusters, 5–30 for noisy data). Delete k-shift edges when at least one endpoint is an outlier, defined as an instance with fewer than -l (default 3) mutual nearest neighbors. The resulting edge-weighted graph is embedded via the Kamada–Kawai force layout, with edge counts across classes visualized as convex-hull links in img_4_hull and img_5_hull_link outputs.

## Related tools

- **GraphDDP** (Method whose force-based graph layout consumes the knn and density-gradient (k_shift) edges to visualize differentiation pathways) — https://github.com/fabriziocosta/GraphEmbed
- **GraphEmbed** (Command-line implementation (graph_embed.py) that constructs the knn/k_shift edges and computes the 2D embedding; installable via PyPI or conda (graph_embed -c bioconda)) — https://github.com/fabriziocosta/GraphEmbed

## Examples

```
./graph_embed.py -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 -l 3 --random_state 2 --normalization --correlation_transformation --feature_selection
```

## Evaluation signals

- img_2_edges.pdf shows red knn edges confined to same-class neighborhoods and blue k-shift edges crossing toward denser different-class regions
- img_5_hull_link.pdf shows class convex hulls connected by first shift-edges reflecting plausible lineage transitions (e.g. HSC-centered branches in the mouse HSPC data)
- The number of emitted k-shift edges respects the -z horizon and -d settings, and outlier endpoints (fewer than -l mutual neighbors) have their k-shift edges removed
- Same-class clusters appear compact in the 2D layout proportional to the chosen -c class confidence (0.5–1 for clean, 5–30 for noisy data)
- The output directory contains 2D_coords.txt, target.txt, and all five img_*.pdf files plus the execution log

## Limitations

- No quantitative performance metrics are reported for the resulting embeddings; validation is visual/qualitative (accepted if images show the expected branching hierarchy)
- Edge quality depends on the density definition (sum of pairwise cosine similarity) and the -z/-l parameters; too small a horizon or outlier threshold can delete meaningful trajectory edges
- Discrete class labels discretize a continuous differentiation process, which is especially problematic for rare cell types such as stem cells

## Evidence

- [abstract] two types of carefully constructed edges: one emphasizing cluster membership, the other, based on density gradients, emphasizing differentiation: "uses a force-based graph layout approach on two types of carefully constructed edges: one emphasizing cluster membership, the other, based on density gradients, emphasizing differentiation"
- [readme] k_shift edge definition: "A k_shift edge is an edge to the k-th nearest instance that is denser and has a different label."
- [readme] density definition: "Density is defined as the sum of the pairwise cosine similarity between an instance and all the other instances."
- [readme] outlier deletion rule: "A k-shift edge is deleted if at least one of the endpoints is an outlier. Outlier nodes are defined as those instances that have no mutual k neighbors."
- [methods] -z horizon parameter: "Number of nearest neighbors to limit the horizon where to search for denser neighbors of a different class [default: 5]"
