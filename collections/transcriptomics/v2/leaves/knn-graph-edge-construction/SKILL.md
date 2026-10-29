---
name: knn-graph-edge-construction
description: Use when a labeled single-cell or feature matrix must be turned into
  a graph whose edges capture both same-class proximity and cross-class density-gradient
  transitions prior to force-based 2D embedding.
when_to_use_negative:
- When no prior cluster/class labels are available — both edge types depend on a user-defined
  label per instance
- When a continuous pseudotime ordering (not a class-informed trajectory graph) is
  the primary goal — pseudotime methods like TSCAN or Monocle2 target that directly
- When instance labels are noisy and compact per-class layouts cannot be forced —
  the -c separation strength (0.5-30) assumes labels are at least partially trusted
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: GraphEmbed
  role: Implements the knn/k_shift graph construction and the force-based layout used
    by GraphDDP
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: graph_embed
  role: Python2/conda command-line entry point (bioconda) executing the same edge-construction
    pipeline
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: pygraphviz
  role: Non-pip dependency required for graph construction/rendering
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:2b3679ec27be7c0cf82d0ad862d416dadbb40b9b361a4462c3c851fc1759431a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fabriziocosta/GraphEmbed
  tools:
  - GraphDDP
  - GraphEmbed
  - graph_embed
  - pygraphviz
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# knn-graph-edge-construction

## Summary

Constructs the two edge types ('knn' and 'k_shift') used by GraphDDP/GraphEmbed to encode cluster membership and density-gradient differentiation pathways in a graph that is then laid out with the Kamada-Kawai force algorithm.

## When to use

Use when a labeled single-cell or feature matrix must be turned into a graph whose edges capture both same-class proximity and cross-class density-gradient transitions prior to force-based 2D embedding.

## When NOT to use

- When no prior cluster/class labels are available — both edge types depend on a user-defined label per instance
- When a continuous pseudotime ordering (not a class-informed trajectory graph) is the primary goal — pseudotime methods like TSCAN or Monocle2 target that directly
- When instance labels are noisy and compact per-class layouts cannot be forced — the -c separation strength (0.5-30) assumes labels are at least partially trusted

## Inputs

- CSV data matrix (e.g. example/prot_expression.csv) with one instance per row
- CSV class/cluster label file (e.g. example/target.csv), one discrete label per instance
- Parameters: -k (same-class neighbors, default 3), -d (denser different-class links, default 1), -z (search horizon, default 5), -l (mutual-NN outlier threshold, default 3), -c (class_confidence, default 1.0)

## Outputs

- Combined knn + k_shift edge set on instance nodes
- Outlier node set (instances without -l mutual nearest neighbors)
- 2D_coords.txt with one 2D embedding coordinate per instance
- img_2_edges.pdf rendering knn edges in red and k-shift edges in blue

## How to apply

After preprocessing the data matrix (optionally with --normalization, --correlation_transformation, --feature_selection), compute per-instance density as the sum of pairwise cosine similarities between an instance and all other instances, and flag outliers as instances lacking the required number of mutual nearest neighbors (parameter -l, default 3; set -l 0 to disable outlier pruning). Build 'knn' edges by linking each instance to its k-th nearest instance sharing the same label (parameter -k, default 3), excluding outlier endpoints, and scale the desired edge length by 1/(1 + class_confidence) for same-label edges (parameter -c, default 1.0; use 0.5-1 for clean clusters and 5-30 for noisy data). Build 'k_shift' edges by linking each instance to its d nearest neighbors that are denser and have a different label, searching only within the horizon of the z nearest neighbors (parameters -d default 1, -z default 5); delete k-shift edges if at least one endpoint is an outlier. The desired edge length is the Euclidean distance between instances, and the combined edge set is passed to the Kamada-Kawai force layout to produce 2D coordinates.

## Related tools

- **GraphEmbed** (Implements the knn/k_shift graph construction and the force-based layout used by GraphDDP) — https://github.com/fabriziocosta/GraphEmbed
- **graph_embed** (Python2/conda command-line entry point (bioconda) executing the same edge-construction pipeline) — https://github.com/fabriziocosta/GraphEmbed
- **pygraphviz** (Non-pip dependency required for graph construction/rendering)

## Examples

```
./graph_embed.py -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 -l 0 --random_state 2 --normalization --correlation_transformation --feature_selection
```

## Evaluation signals

- 2D_coords.txt contains one coordinate line per input instance and target.txt one label line per instance
- img_2_edges.pdf exists as a valid PDF and renders knn edges in red and k-shift edges in blue
- Edge-endpoint invariants: every knn edge connects same-label instances (scaled by 1 + class_confidence), every k_shift edge connects a denser instance to a different-label one
- No surviving edge has an outlier endpoint (k-shift edges are deleted when at least one endpoint has fewer than -l mutual nearest neighbors)
- On the intestinal epithelial reference run, the layout reproduces the Fig. 2c topology: stem-cell cluster 7 centered with lineage trajectories attached

## Limitations

- Outlier instances (fewer than -l mutual nearest neighbors) lose their k-shift edges, so rare cells may be disconnected from trajectories
- Edge quality depends on cosine-similarity density estimates; -c separation strength must be tuned (0.5-1 clean, 5-30 noisy) per dataset
- No quantitative metrics in the article validate the resulting layouts — assessment is expert visual comparison (e.g. against Fig. 2c)

## Evidence

- [readme] knn and k_shift edge definitions: "A knn edge is an edge to the k-th nearest instance that has the same label. A k_shift edge is an edge to the k-th nearest instance that is denser and has a different label."
- [readme] density definition and outlier rule: "Density is defined as the sum of the pairwise cosine similarity between an instance and all the other instances."
- [readme] outlier and same-label distance handling: "If the endpoints of an edge have the same label then the desired distance is divided by 1 + class_confidence."
- [readme] layout algorithm: "the embedding is computed as the 2D coordinates of the corresponding graph embedding using the force layout algorithm from Tomihisa Kamada"
- [abstract] two edge types in the article: "two types of carefully constructed edges: one emphasizing cluster membership, the other, based on density gradients, emphasizing differentiation"
