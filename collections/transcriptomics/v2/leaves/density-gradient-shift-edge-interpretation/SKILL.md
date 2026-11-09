---
name: density-gradient-shift-edge-interpretation
description: Use when a graph-embedding visualization with prior class labels must
  reveal continuous differentiation orderings between clusters, e.g. resolving meta-cluster
  C7-C1 in myeloid progenitor single-cell data.
when_to_use_negative:
- When the goal is pseudotime per-cell ordering rather than cluster-level pathway
  visualization — tools like TSCAN, Monocle2 or Wishbone build MST/pseudotime trajectories
  instead of shift-edge layouts
- When no prior class/cluster labels exist for the instances, since both knn and k_shift
  edges are defined relative to class labels
- 'When class hulls are naturally well separated (clean data): the README says values
  of 0.5-1 for -c suit clean data; forcing -c 5-30 is meant for noisy data needing
  strong separation'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: GraphDDP
  role: Method whose force-based layout on knn (cluster-membership) and density-gradient
    k_shift (differentiation) edges produces the visualization being interpreted
  repo: github.com/fabriziocosta/GraphEmbed
- name: graph_embed
  role: CLI implementation (python2/bioconda) that computes the 2D embedding and emits
    the img_1 through img_5 PDFs and 2D_coords.txt inspected during interpretation
  repo: https://bioconda (conda install graph_embed -c bioconda)
- name: GraphEmbed
  role: PyPI-installable package providing graph_embed.py used to generate the embedding
    outputs
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: pygraphviz
  role: Non-pip graphviz dependency required to draw the embedding figures with knn
    and shift edges
  repo: https://github.com/fabriziocosta/GraphEmbed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:a6d9e9dff72077196c4de61bf80052b910d3f8af807f4052ab0e4d2a3e5ad23b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphDDP
  - GraphEmbed
  - graph_embed
  - pygraphviz
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# density-gradient-shift-edge-interpretation

## Summary

Interprets the k_shift (density-gradient) edges of a GraphDDP/GraphEmbed 2D layout to order classes along differentiation pathways, using cross-class shift-edge counts and ternary confusion scores (e.g. 0.06 for resolved C5-C6-C7 vs 0.72 for unresolved C2-C3-C4) in the Paul et al. myeloid progenitor data.

## When to use

Use when a graph-embedding visualization with prior class labels must reveal continuous differentiation orderings between clusters, e.g. resolving meta-cluster C7-C1 in myeloid progenitor single-cell data.

## When NOT to use

- When the goal is pseudotime per-cell ordering rather than cluster-level pathway visualization — tools like TSCAN, Monocle2 or Wishbone build MST/pseudotime trajectories instead of shift-edge layouts
- When no prior class/cluster labels exist for the instances, since both knn and k_shift edges are defined relative to class labels
- When class hulls are naturally well separated (clean data): the README says values of 0.5-1 for -c suit clean data; forcing -c 5-30 is meant for noisy data needing strong separation

## Inputs

- Gene/protein expression matrix CSV (e.g. prot_expression.csv, one row per cell)
- Class-assignment (target) CSV with a discrete label per instance (e.g. Paul et al. classes C1-C9)
- graph_embed.py output directory containing 2D_coords.txt, target.txt, and img_1_clean.pdf through img_5_hull_link.pdf

## Outputs

- Per-class adjacency description of first shift-edges revealing differentiation transitions (e.g. C7→C6→C5 within the erythrocyte meta-cluster C7-C1)
- Ternary-plot confusion scores quantifying ordering resolution per class triple (0.06 for C5-C6-C7; 0.72 for C2-C3-C4)
- Validated hull/edge figures (img_4_hull.pdf, img_5_hull_link.pdf) matching the published Fig. 3 topology

## How to apply

Run graph_embed.py with the expression matrix (-i), class-assignment CSV (-t), and filters --normalization, --correlation_transformation, --feature_selection; in the noisy myeloid progenitor regime use a strong class confidence in the 5-30 range (e.g. -c 10), keeping defaults -k 3, -d 1, -z 5, -l 3 and fixing --random_state. The graph contains two edge types: red knn edges linking the k-th nearest same-label instance, and blue k_shift edges linking the k-th nearest denser instance with a different label, where density is the sum of pairwise cosine similarity to all other instances; k_shift edges to outliers (instances without mutual k neighbors, -l 3) are deleted. Inspect img_3_edge.pdf (first shift-edges), img_4_hull.pdf and img_5_hull_link.pdf (class convex hulls with shift-edges): a clear class-to-class transition path indicates a resolved ordering, while diffuse or bidirectional shift-edges indicate no ordering. Quantify transitions via ternary-plot confusion scores: C5-C6-C7 = 0.06 means a clean C7→C6→C5 transition, whereas C2-C3-C4 = 0.72 means the three clusters are mutually linked with no resolved direction.

## Related tools

- **GraphDDP** (Method whose force-based layout on knn (cluster-membership) and density-gradient k_shift (differentiation) edges produces the visualization being interpreted) — github.com/fabriziocosta/GraphEmbed
- **graph_embed** (CLI implementation (python2/bioconda) that computes the 2D embedding and emits the img_1 through img_5 PDFs and 2D_coords.txt inspected during interpretation) — https://bioconda (conda install graph_embed -c bioconda)
- **GraphEmbed** (PyPI-installable package providing graph_embed.py used to generate the embedding outputs) — https://github.com/fabriziocosta/GraphEmbed
- **pygraphviz** (Non-pip graphviz dependency required to draw the embedding figures with knn and shift edges) — https://github.com/fabriziocosta/GraphEmbed

## Examples

```
./graph_embed.py -i example/prot_expression.csv -t example/target.csv -c 10 -k 3 -d 1 -z 5 -l 3 --normalization --correlation_transformation --feature_selection --random_state 2
```

## Evaluation signals

- 2D_coords.txt contains exactly one coordinate line per input instance and target.txt one label per instance
- The class convex-hull layout (img_4_hull.pdf / img_5_hull_link.pdf) recovers the erythrocyte meta-cluster C7-C1 adjacency as in Fig. 3 of the paper
- Ternary-plot confusion scores reproduce 0.06 for the C5-C6-C7 triple (clear C7→C6→C5 transition) and 0.72 for C2-C3-C4 (no resolved ordering)
- img_2_edges.pdf shows red knn edges and blue k_shift edges consistent with the specified -k, -d, -z parameters and outlier deletion at -l 3
- Repeated runs with a fixed --random_state produce reproducible hull/edge figures

## Limitations

- Class-confidence -c controls separation: values 0.5-1 suit clean data and 5-30 suit noisy data, so an inappropriate -c can distort the shift-edge topology being interpreted
- k_shift edges are deleted when either endpoint is an outlier (no mutual -l nearest neighbors), so very sparse subpopulations may lose their transition edges entirely
- The paper reports no quantitative performance metrics versus t-SNE, PCA, or MDS; superiority is argued qualitatively from the visualized hierarchies
- No dataset accessions are cited for the myeloid progenitor data, and the compared tools (TSCAN, Monocle2, SPRING) lack documented versions or parameter settings

## Evidence

- [readme] A knn edge is an edge to the k-th nearest instance that has the same label. A k_shift edge is an edge to the k-th nearest instance that is denser and has a different label.: "A knn edge is an edge to the k-th nearest instance that has the same label.
A k_shift edge is an edge to the k-th nearest instance that is denser
and has a different label."
- [readme] Density defined as cosine similarity sum; outliers deleted.: "Density is defined as the sum of the pairwise cosine similarity between
an instance and all the other instances."
- [readme] Values of 5-30 are suitable for noisy data where it is necessary to force a strong separation.: "values of 5-30 are suitable for noisy data where it is necessary to force a strong separation in the 2D representation."
- [abstract] Abstract: force-based layout on two edge types, one emphasizing cluster membership, the other, based on density gradients, emphasizing differentiation.: "uses a force-based graph layout approach on two types of carefully constructed edges: one emphasizing cluster membership, the other, based on density gradients, emphasizing differentiation"
- [methods] Methods: setting separation strength and neighbor counts.: "To set a desired separation strength use -c, to specify the number of neighbors use -k"
- [discussion] Discussion: GraphDDP detected trajectories not identified by t-SNE, PCA or MDS.: "We found that GraphDDP could detect differentiation trajectories in intestinal epithelial and myeloid progenitor cells that could not be identified with state-of-the-art visualization approaches such"
