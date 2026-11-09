---
name: quickshift-density-gradient-edge-computation
description: Use when a supervised 2D embedding must reveal differentiation pathways
  across a prior cell-type clustering by linking each instance to denser, differently-labeled
  neighbors (e.g. scRNA-seq data with cluster assignments).
when_to_use_negative:
- All instances share the same class label — no denser different-label neighbors exist,
  so no k-shift edges can be formed
- The goal is pure cluster-membership visualization without differentiation-trajectory
  structure — the 'knn' same-label edges alone suffice
- Class assignments are unreliable placeholders rather than prior cluster knowledge,
  since edge direction and deletion depend on label differences
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: GraphEmbed
  role: Python tool that builds the k_shift edges from density (summed pairwise cosine
    similarity) and label information, and computes the 2D force-based embedding
  repo: github.com/fabriziocosta/GraphEmbed
- name: graph_embed
  role: Conda/bioconda distribution of the same graph_embed.py pipeline (python2)
    that executes the k-shift edge construction with -d/-z/-l parameters
  repo: bioconda
- name: pygraphviz
  role: Non-pip graphviz dependency needed to install GraphEmbed and handle the constructed
    graph
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:6deac6e4f9e7a75bb9ad76be22a7f167237f249604a3727aabe88a65912f657e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphDDP
  - GraphEmbed
  - graph_embed
  - pygraphviz
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# quickshift-density-gradient-edge-computation

## Summary

Computes the 'k_shift' edges of the GraphDDP/GraphEmbed graph: each instance is linked to its d nearest neighbors that are denser (density = sum of pairwise cosine similarities) and carry a different class label. These quick-shift-style density-gradient edges encode candidate differentiation trajectories that the force-based (Kamada-Kawai) 2D layout renders in blue.

## When to use

Use when a supervised 2D embedding must reveal differentiation pathways across a prior cell-type clustering by linking each instance to denser, differently-labeled neighbors (e.g. scRNA-seq data with cluster assignments).

## When NOT to use

- All instances share the same class label — no denser different-label neighbors exist, so no k-shift edges can be formed
- The goal is pure cluster-membership visualization without differentiation-trajectory structure — the 'knn' same-label edges alone suffice
- Class assignments are unreliable placeholders rather than prior cluster knowledge, since edge direction and deletion depend on label differences

## Inputs

- Data matrix in CSV format (e.g. example/prot_expression.csv), optionally normalized or correlation-transformed
- Class assignment file in CSV format (e.g. example/target.csv), one discrete label per instance
- Parameters: -d (number of links to denser different-class neighbors, default 1), -z (search horizon in nearest neighbors, default 5), -l (mutual-neighbor outlier threshold, default 3)

## Outputs

- Set of k_shift edges (source instance -> denser, different-label instance) used by the force-based layout
- img_2_edges.pdf with k-shift edges drawn as blue lines (and knn edges in red)
- 2D_coords.txt (2D coordinates, one line per instance) and target.txt written to the timestamped output directory
- Program execution log file

## How to apply

Load the data matrix (e.g. example/prot_expression.csv, optionally transformed via --normalization or --correlation_transformation) and the class labels from the target CSV (e.g. example/target.csv). Estimate per-instance density as the sum of pairwise cosine similarity between an instance and all other instances, then for each instance search only within the horizon of its z nearest neighbors (parameter -z, default 5) for instances that are denser and have a different label, and add d links towards them (parameter -d, default 1). Before edge insertion, flag outliers as instances lacking the required number of mutual nearest neighbors (parameter -l, default 3) and delete any k-shift edge with at least one outlier endpoint. Pass the resulting edge set, together with the 'knn' same-label edges, to the Kamada-Kawai force-based graph layout to obtain 2D coordinates, and render k-shift edges as blue lines in img_2_edges.pdf (knn edges in red) in the timestamped -o output directory.

## Related tools

- **GraphEmbed** (Python tool that builds the k_shift edges from density (summed pairwise cosine similarity) and label information, and computes the 2D force-based embedding) — github.com/fabriziocosta/GraphEmbed
- **graph_embed** (Conda/bioconda distribution of the same graph_embed.py pipeline (python2) that executes the k-shift edge construction with -d/-z/-l parameters) — bioconda
- **pygraphviz** (Non-pip graphviz dependency needed to install GraphEmbed and handle the constructed graph)

## Examples

```
./graph_embed.py -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 -l 0 --random_state 2
```

## Evaluation signals

- Every non-outlier instance has at most d (default 1) k-shift edges, each pointing to a strictly denser instance with a different label, found within the z (default 5) nearest-neighbor horizon
- No k-shift edge has an outlier endpoint (outliers = instances without the required number of mutual nearest neighbors per -l)
- img_2_edges.pdf exists as a valid PDF and visually distinguishes blue k-shift edges from red knn edges
- 2D_coords.txt contains one line per instance, matching the number of rows in the input data matrix, with corresponding target.txt labels
- All output files (img_1..img_5 pdf, coords, targets, log) appear in the directory named by -o with the year-month-day-hour-min-sec timestamp suffix

## Limitations

- Instances flagged as outliers (no mutual k neighbors, -l default 3) lose their k-shift edges, so trajectories through sparse regions are not represented
- Density is defined only via summed pairwise cosine similarity between an instance and all others; other density estimators are not used
- The article reports no quantitative performance metrics supporting superiority over t-SNE, PCA, or MDS — evaluation is qualitative (visual trajectory detection on intestinal epithelial and myeloid progenitor data)

## Evidence

- [abstract] k-shift edges emphasize differentiation via density gradients in the force-based layout: "uses a force-based graph layout approach on two types of carefully constructed edges: one emphasizing cluster membership, the other, based on density gradients, emphasizing differentiation"
- [readme] Definition of the k_shift edge: "A k_shift edge is an edge to the k-th nearest instance that is denser and has a different label."
- [readme] Density definition used for edge direction: "Density is defined as the sum of the pairwise cosine similarity between an instance and all the other instances."
- [readme] Outlier deletion rule for k-shift edges: "A k-shift edge is deleted if at least one of the endpoints is an outlier."
- [readme] Search-horizon and link-count parameters: "-z N Number of nearest neighbors to limit the horizon where to search for denser neighbors of a different class [default: 5]"
