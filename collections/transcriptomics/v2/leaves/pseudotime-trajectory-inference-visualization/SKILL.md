---
name: pseudotime-trajectory-inference-visualization
description: Use when evaluating whether pseudotime-based approaches such as TSCAN
  can identify differentiation pathways on single-cell RNA-seq data given the same
  number of clusters as a class-informed method, or when comparing trajectory inference
  to visualization baselines.
when_to_use_negative:
- When the goal is a class-informed visualization integrating prior cluster assignments
  — GraphDDP's force-based layout with -c class_confidence is the appropriate tool
  instead
- When only a dimensionality-reduction baseline (t-SNE, PCA, MDS) is needed without
  trajectory inference
- When quantitative benchmarking metrics are required — the paper reports only qualitative,
  expert-review validation for these baselines
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: TSCAN
  role: 'Pseudotime reconstruction baseline: clusters cells, builds a minimum spanning
    tree between cluster centers, and projects cells onto the tree to obtain pseudotime'
  repo: https://github.com/zji90/TSCAN
- name: GraphEmbed
  role: GraphDDP implementation used to supply the class-informed embedding and the
    target class counts (6 intestinal, 20 myeloid) that TSCAN is matched against
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: pygraphviz
  role: Non-pip dependency required before installing GraphEmbed via PyPI
- name: SPRING
  role: k-nearest-neighbor force-layout comparison baseline (-k 3, -k 7, optional
    -c class contraction) run alongside the TSCAN pseudotime analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:a9c6ee00b0fb6ff1f03b9887900758128479c59200a224241e3791b97787b0d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zji90/TSCAN
  tools:
  - GraphDDP
  - GraphEmbed
  - SPRING
  - TSCAN
  - pygraphviz
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# pseudotime-trajectory-inference-visualization

## Summary

Run TSCAN cluster-based minimum-spanning-tree pseudotime inference as a baseline against graph-embedding visualizations like GraphDDP, overlaying the inferred pseudotime on intestinal epithelial and myeloid progenitor single-cell data to test whether trajectory-only methods recover differentiation pathways.

## When to use

Use when evaluating whether pseudotime-based approaches such as TSCAN can identify differentiation pathways on single-cell RNA-seq data given the same number of clusters as a class-informed method, or when comparing trajectory inference to visualization baselines.

## When NOT to use

- When the goal is a class-informed visualization integrating prior cluster assignments — GraphDDP's force-based layout with -c class_confidence is the appropriate tool instead
- When only a dimensionality-reduction baseline (t-SNE, PCA, MDS) is needed without trajectory inference
- When quantitative benchmarking metrics are required — the paper reports only qualitative, expert-review validation for these baselines

## Inputs

- Expression matrix CSV for intestinal epithelial cells dataset
- Expression matrix CSV for myeloid progenitor dataset
- Class/target CSV files with cluster labels (6 classes intestinal, 20 classes myeloid)

## Outputs

- TSCAN cluster-MST trajectory with pseudotime values projected onto each cell
- Overlay plots showing pseudotime within clusters and class-splitting patterns (cf. Supplementary Figs. 7-8)

## How to apply

Install TSCAN from GitHub in R (via devtools::install_github('zji90/TSCAN')). Load the expression matrix and class/target CSV files for the intestinal epithelial and myeloid progenitor datasets, then run TSCAN with the same number of clusters as the GraphDDP classes (6 clusters for intestinal data, 20 clusters for myeloid data) using its cluster-based MST approach: cells are first clustered, an MST connects cluster centers, and pseudotime is obtained by projecting each cell onto the tree. Overlay the inferred pseudotime on the cluster assignments to examine whether major differentiation pathways are visible and whether clusters split relative to the original labels. Accept outputs by expert review of qualitative patterns, since no quantitative metric is reported in the paper.

## Related tools

- **TSCAN** (Pseudotime reconstruction baseline: clusters cells, builds a minimum spanning tree between cluster centers, and projects cells onto the tree to obtain pseudotime) — https://github.com/zji90/TSCAN
- **GraphEmbed** (GraphDDP implementation used to supply the class-informed embedding and the target class counts (6 intestinal, 20 myeloid) that TSCAN is matched against) — https://github.com/fabriziocosta/GraphEmbed
- **pygraphviz** (Non-pip dependency required before installing GraphEmbed via PyPI)
- **SPRING** (k-nearest-neighbor force-layout comparison baseline (-k 3, -k 7, optional -c class contraction) run alongside the TSCAN pseudotime analysis)

## Examples

```
devtools::install_github("zji90/TSCAN"); library(TSCAN); proc <- exprmclust(expr_matrix, cluster_num=20); TSCANorder(proc, return_state=TRUE)
```

## Evaluation signals

- On myeloid progenitor data with 20 clusters, TSCAN detects only a single linear pathway rather than multiple major differentiation pathways
- Large pseudotime differences are observed within individual clusters, and cluster splitting occurs relative to the original class labels
- The qualitative patterns match Supplementary Figs. 7-8 (single-pathway detection and class splitting) upon expert review
- The number of TSCAN clusters equals the number of GraphDDP classes (6 for intestinal, 20 for myeloid), ensuring a fair comparison
- Pseudotime overlays can be visually inspected against the GraphDDP embedding to confirm pathways visible in GraphDDP are missed by TSCAN

## Limitations

- No quantitative performance metric is reported in the paper; validation of the reproduced baseline is by expert review of qualitative patterns only
- No software versions or parameter settings for TSCAN beyond cluster counts are documented in the paper
- No dataset accessions or named data resources are cited for the intestinal epithelial and myeloid progenitor datasets

## Evidence

- [results] TSCAN detects only a single linear pathway on the myeloid data even when given the same number of classes as GraphDDP (20): [evidence span withheld — anchor status: not_found]
- [readme] TSCAN uses a cluster-based minimum spanning tree (MST) approach to order cells: [evidence span withheld — anchor status: external_doc]
- [discussion] including pseudotime-based methods like TSCAN and Monocle2: "including pseudotime-based methods like TSCAN and Monocle2, as well as k-nearest neighbor approaches like SPRING"
- [supplementary] Run TSCAN on the intestinal data with 6 clusters and on the myeloid data with 20 clusters: [evidence span withheld — anchor status: not_found]
