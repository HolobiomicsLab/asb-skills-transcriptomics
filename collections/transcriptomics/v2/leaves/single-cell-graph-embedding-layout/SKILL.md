---
name: single-cell-graph-embedding-layout
description: Use when you have a scRNA-seq expression matrix plus prior cluster labels
  and need a 2D layout that shows cluster membership together with differentiation
  trajectories between clusters (e.g. stem-cell-centered lineages, branch points).
when_to_use_negative:
- No prior cluster assignment exists — GraphDDP starts from a user-defined cluster
  label per instance; run a clustering step (e.g. RaceID2) first.
- You need quantitative pseudotime ordering or a rooted trajectory tree rather than
  a visual layout — use pseudotime tools like TSCAN or Monocle2 instead.
- The input is not an instance/feature matrix with discrete labels (e.g. already a
  feature table of summary statistics or unlabeled bulk data).
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: GraphEmbed
  role: PyPI-installable implementation of the supervised 2D graph embedding (graph_embed.py)
    that builds knn/k-shift edges and computes the Kamada–Kawai force layout
  repo: github.com/fabriziocosta/GraphEmbed
- name: pygraphviz
  role: Non-pip dependency installed via conda to support the graph layout computation
- name: graph_embed
  role: Conda/bioconda distribution of the python2 version of the embedding tool
  repo: github.com/fabriziocosta/GraphEmbed
- name: TSCAN
  role: Alternative cluster-based MST pseudotime method; not used within this skill
    but discussed as a comparison approach
  repo: zji90/TSCAN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:ea4cf177000ee6b7364745fe77891483f5b336f046a21120d37b82e6eb145159
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphDDP
  - GraphEmbed
  - TSCAN
  - graph_embed
  - pygraphviz
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-graph-embedding-layout

## Summary

Compute a supervised 2D graph embedding of single-cell transcriptome data (GraphDDP/GraphEmbed) that integrates RaceID2/StemID-style cluster assignments with density-gradient differentiation trajectories, so lineage hierarchies and progenitor relations become visible in one layout.

## When to use

Use when you have a scRNA-seq expression matrix plus prior cluster labels and need a 2D layout that shows cluster membership together with differentiation trajectories between clusters (e.g. stem-cell-centered lineages, branch points).

## When NOT to use

- No prior cluster assignment exists — GraphDDP starts from a user-defined cluster label per instance; run a clustering step (e.g. RaceID2) first.
- You need quantitative pseudotime ordering or a rooted trajectory tree rather than a visual layout — use pseudotime tools like TSCAN or Monocle2 instead.
- The input is not an instance/feature matrix with discrete labels (e.g. already a feature table of summary statistics or unlabeled bulk data).

## Inputs

- Input data matrix as CSV (e.g. scRNA-seq expression matrix, one row per cell, such as the 462-cell intestinal epithelial dataset)
- Target CSV with one discrete class label per instance (e.g. RaceID2 cluster assignments)
- Optional parameters: -c class separation strength, -k same-class neighbors, -d denser different-class links, -z search horizon, -l mutual-neighbor outlier threshold

## Outputs

- 2D_coords.txt — 2D coordinates, one line per instance
- target.txt — target identifier, one line per instance
- img_1_clean.pdf through img_5_hull_link.pdf — embedding images (clean layout, knn/k-shift edges, first shift-edges, class convex hulls with cross-class shift-edge counts)
- log — program execution log, all placed in a timestamped output directory

## How to apply

Install the embedding environment with pygraphviz as the non-pip dependency (conda install pygraphviz) and GraphEmbed via PyPI (pip install graphembed) or the python2 bioconda channel (conda install graph_embed -c bioconda). Prepare two CSVs: the input data matrix (e.g. the 462-cell intestinal epithelial scRNA-seq matrix) and a target file with one discrete class label (RaceID2 cluster id) per instance. Run graph_embed.py enabling --normalization, --correlation_transformation (converts the matrix to a correlation coefficient matrix) and --feature_selection (selects most discriminative features); the graph is built from 'knn' edges to same-label nearest neighbors (-k, default 3) and 'k_shift' edges to denser, different-label neighbors (-d, default 1) within a search horizon (-z, default 5), with outliers defined as instances lacking mutual neighbors (-l, default 3). Set the class separation strength via -c: 0.5–1 for clean, well-separated clusters, 5–30 for noisy data requiring forced separation. The Kamada–Kawai force layout yields the 2D coordinates; inspect img_1_clean.pdf and the edge/hull overlays (img_2 through img_5) to read off trajectories as chains of shift-edges between cluster hulls.

## Related tools

- **GraphEmbed** (PyPI-installable implementation of the supervised 2D graph embedding (graph_embed.py) that builds knn/k-shift edges and computes the Kamada–Kawai force layout) — github.com/fabriziocosta/GraphEmbed
- **pygraphviz** (Non-pip dependency installed via conda to support the graph layout computation)
- **graph_embed** (Conda/bioconda distribution of the python2 version of the embedding tool) — github.com/fabriziocosta/GraphEmbed
- **TSCAN** (Alternative cluster-based MST pseudotime method; not used within this skill but discussed as a comparison approach) — zji90/TSCAN

## Examples

```
./graph_embed.py -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 -l 0 --normalization --correlation_transformation --feature_selection
```

## Evaluation signals

- Output directory exists with the expected files: 2D_coords.txt, target.txt, img_1_clean.pdf through img_5_hull_link.pdf, and log, with a timestamp suffix
- 2D_coords.txt has exactly one coordinate line per input instance and target.txt has one label line per instance, in matching order
- On the intestinal epithelial dataset (462 cells, RaceID2 clusters), img_1_clean.pdf shows stem cell cluster 7 centered and distinct lineages assembled on connected trajectories matching the reported Fig. 2c topology (expert visual review)
- Cluster 1 appears as a common progenitor of Paneth and goblet cells, and the enteroendocrine branches separate into Cck+/Sst+ versus Gip+/Tac1+ arms
- Convex hull overlays (img_4/img_5) show compact same-class hulls and cross-class shift-edge thickness reflecting the number of first shift-edges between classes

## Limitations

- No quantitative performance metrics are reported supporting superiority over t-SNE, PCA, or MDS — validation relies on visual/expert inspection of the embedding images.
- No software versions or parameter settings for the compared tools (TSCAN, Monocle2, SPRING) are given, complicating reproduction of the comparison.
- No dataset accessions or named data resources are cited for the intestinal epithelial and myeloid progenitor datasets.
- Class separation requires the right -c regime: 0.5–1 only suits clean, well-separated clusters; noisy data needs 5–30, and mis-setting it degrades the layout.
- k-shift edges are deleted when an endpoint is an outlier (no mutual k neighbors), so rare isolated cells may lose trajectory links.

## Evidence

- [abstract] Supervised layout integrating clusters and trajectories: "uses a force-based graph layout approach on two types of carefully constructed edges: one emphasizing cluster membership, the other, based on density gradients, emphasizing differentiation"
- [abstract] Graph starts from user-defined cluster assignment: "GraphDDP starts from a user-defined cluster assignment and then uses a force-based graph layout approach"
- [readme] README edge construction rules: "A knn edge is an edge to the k-th nearest instance that has the same label. A k_shift edge is an edge to the k-th nearest instance that is denser and has a different label."
- [readme] Outlier definition: "Outlier nodes are defined as those instances that have no mutual k neighbors."
- [methods] Class confidence regime guidance: "Values of 0.5-1 are suitable for clean data where clusters are naturally well separated, values of 5-30 are suitable for noisy data"
- [readme] Layout algorithm citation: "the embedding is computed as the 2D coordinates of the corresponding graph embedding using the force layout algorithm from Tomihisa Kamada, and Satoru Kawai."
- [readme] Density definition: "Density is defined as the sum of the pairwise cosine similarity between an instance and all the other instances."
