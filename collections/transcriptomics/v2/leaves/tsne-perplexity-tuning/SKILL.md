---
name: tsne-perplexity-tuning
description: Use when benchmarking a supervised or class-informed embedding (e.g.
  GraphDDP) against unsupervised baselines and you must demonstrate that t-SNE's perplexity
  choice can erase cluster/class structure on labeled single-cell data.
when_to_use_negative:
- When the goal is the supervised GraphDDP embedding itself rather than the unsupervised
  baselines — use the class-contraction settings (-c) of GraphEmbed instead
- When quantitative accuracy metrics are required — no quantitative metric is reported
  for these baseline comparisons in the paper
- 'When class labels are unavailable: t-SNE, SPRING k-NN-only layouts, PCA, and MDS
  are unsupervised and this skill''s comparison is about demonstrating their class-blindness'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GraphEmbed
  role: Provides the graph-embedding CLI used to generate the k-NN force layouts and
    class-contracted embeddings that are compared against t-SNE baselines
  repo: github.com/fabriziocosta/GraphEmbed
- name: pygraphviz
  role: Non-pip graphviz dependency required before installing GraphEmbed via PyPI
- name: TSCAN
  role: Cluster-based MST pseudotime method run with matched cluster counts (6 and
    20) as a trajectory-inference baseline
  repo: zji90/TSCAN
- name: SPRING
  role: k-nearest-neighbor force-layout baseline whose k-NN-only variants (-k 3, -k
    7) are reproduced without class contraction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:aba21f6819179ed40aa71fae1600cc9faf73ce269e0795092b25eab0b538b966
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphDDP
  - GraphEmbed
  - SPRING
  - TSCAN
  - pygraphviz
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# tsne-perplexity-tuning

## Summary

Reproduces the unsupervised baseline comparisons from the GraphDDP paper: t-SNE with elevated perplexity on intestinal epithelial and myeloid progenitor single-cell data to show loss of class separation, alongside SPRING-style k-NN force layouts and TSCAN pseudotime with matched cluster counts, plus PCA/MDS controls.

## When to use

Use when benchmarking a supervised or class-informed embedding (e.g. GraphDDP) against unsupervised baselines and you must demonstrate that t-SNE's perplexity choice can erase cluster/class structure on labeled single-cell data.

## When NOT to use

- When the goal is the supervised GraphDDP embedding itself rather than the unsupervised baselines — use the class-contraction settings (-c) of GraphEmbed instead
- When quantitative accuracy metrics are required — no quantitative metric is reported for these baseline comparisons in the paper
- When class labels are unavailable: t-SNE, SPRING k-NN-only layouts, PCA, and MDS are unsupervised and this skill's comparison is about demonstrating their class-blindness

## Inputs

- Expression matrix CSV for the intestinal epithelial cells dataset
- Expression matrix CSV for the myeloid progenitor dataset
- Class/target CSV files assigning a discrete label per instance

## Outputs

- t-SNE embedding images with elevated perplexity showing lost class separation
- SPRING-style k-NN force layouts (-k 3, -k 7, -k 7 with -c 1.5)
- TSCAN pseudotime overlays on 6-cluster (intestinal) and 20-cluster (myeloid) trees
- PCA and MDS embeddings of the myeloid dataset
- img_1_clean.pdf and 2D_coords.txt coordinate files per embedding in a timestamped -o output directory

## How to apply

Install GraphEmbed via PyPI after installing the pygraphviz dependency with conda. Load the expression matrix and class/target CSV files, then run t-SNE with an elevated perplexity setting on both datasets to reproduce the layout in which class separation is lost. Build SPRING-style k-NN-only force layouts with -k 3 and -k 7 (class_confidence effectively 0), then rerun with -k 7 and a class-contraction prior via -c (e.g. -c 1.5) to isolate the effect of the class edge weighting. Run TSCAN on the intestinal data with 6 clusters and on the myeloid data with 20 clusters, overlaying pseudotime to check whether TSCAN detects only a single linear pathway and splits original class labels. Finally compute PCA and MDS embeddings on the myeloid dataset as further unsupervised controls. The rationale is that each reduction technique has a specific bias — t-SNE depends on the perplexity parameter and can group outliers arbitrarily, and classical approaches ignore class information from a prior clustering phase.

## Related tools

- **GraphEmbed** (Provides the graph-embedding CLI used to generate the k-NN force layouts and class-contracted embeddings that are compared against t-SNE baselines) — github.com/fabriziocosta/GraphEmbed
- **pygraphviz** (Non-pip graphviz dependency required before installing GraphEmbed via PyPI)
- **TSCAN** (Cluster-based MST pseudotime method run with matched cluster counts (6 and 20) as a trajectory-inference baseline) — zji90/TSCAN
- **SPRING** (k-nearest-neighbor force-layout baseline whose k-NN-only variants (-k 3, -k 7) are reproduced without class contraction)

## Examples

```
./graph_embed.py -i example/prot_expression.csv -t example/target.csv -k 7 -c 1.5 -o out
```

## Evaluation signals

- Expert visual review confirms t-SNE class separation is lost at the elevated perplexity setting on both datasets
- TSCAN pseudotime on myeloid data (20 clusters) spans only a single linear pathway with large within-cluster pseudotime differences and visible class splitting relative to original labels
- k-NN-only force layouts (class_confidence effectively 0) show mixed/overlapping clusters, while adding -c 1.5 restores cluster compaction
- Each run exports img_1_clean.pdf together with a 2D_coords.txt file in the timestamped -o output directory

## Limitations

- No quantitative performance metric is reported to support the claimed superiority of GraphDDP over t-SNE, PCA, or MDS — validation is qualitative expert review
- No software versions or parameter settings for TSCAN, Monocle2, or SPRING are given for the comparisons
- No dataset accessions or named data resources are cited for the intestinal epithelial and myeloid progenitor datasets
- t-SNE depends on the perplexity parameter and can group outliers arbitrarily

## Evidence

- [intro] t-SNE depends on the perplexity parameter and can group outliers arbitrarily: [evidence span withheld — anchor status: not_found]
- [methods] GraphEmbed PyPI install: "GraphEmbed can be installed via PyPI"
- [methods] GraphEmbed parameters: "To set a desired separation strength use -c, to specify the number of neighbors use -k"
- [readme] TSCAN MST method: [evidence span withheld — anchor status: external_doc]
- [intro] Unsupervised baselines ignore class info: "the classical dimensionality reduction approaches are unsupervised, e.g. they do not take into account class information available, for example, from a prior clustering phase."
