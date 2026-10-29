---
name: single-cell-dimensionality-reduction-benchmarking
description: Use when benchmarking a supervised graph-embedding visualization (e.g.
  GraphDDP) against unsupervised and pseudotime baselines on single-cell data with
  prior cluster labels, to show which pathways baselines miss.
when_to_use_negative:
- When a quantitative accuracy metric for pseudotime or trajectory recovery is required
  — the paper reports no quantitative metric and validation is by expert visual review
- When the goal is to reproduce GraphDDP itself with a class-contraction prior rather
  than the unsupervised/pseudotime baselines (this skill covers only the baseline
  comparisons)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: GraphEmbed
  role: Executed with -k only (class_confidence effectively 0) to produce SPRING-style
    k-NN force layouts, and with -c 1.5 to add the class-contraction prior for the
    baseline comparison
  repo: github.com/fabriziocosta/GraphEmbed
- name: TSCAN
  role: 'Pseudotime-based baseline: run with 6 clusters on intestinal and 20 clusters
    on myeloid data to test whether pseudotime alone detects the differentiation pathways'
  repo: zji90/TSCAN
- name: SPRING
  role: Conceptual reference for k-NN-only force layouts reproduced via GraphEmbed
    without class contraction
- name: pygraphviz
  role: Non-pip dependency installed via conda before installing GraphEmbed from PyPI
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:9c22a10d846858ae46e09ff8dca1593253c5f7dcd1e702c2935a236b1cd2cb7a
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# single-cell-dimensionality-reduction-benchmarking

## Summary

Reproduce the baseline visualizations (t-SNE at elevated perplexity, SPRING-style k-NN force layouts with and without class contraction, TSCAN pseudotime, PCA, MDS) that the GraphDDP paper uses to show that unsupervised and pseudotime-only approaches miss differentiation pathways in intestinal epithelial and myeloid progenitor single-cell data.

## When to use

Use when benchmarking a supervised graph-embedding visualization (e.g. GraphDDP) against unsupervised and pseudotime baselines on single-cell data with prior cluster labels, to show which pathways baselines miss.

## When NOT to use

- When a quantitative accuracy metric for pseudotime or trajectory recovery is required — the paper reports no quantitative metric and validation is by expert visual review
- When the goal is to reproduce GraphDDP itself with a class-contraction prior rather than the unsupervised/pseudotime baselines (this skill covers only the baseline comparisons)

## Inputs

- Expression matrix CSV for the intestinal epithelial cells dataset (e.g. prot_expression.csv format)
- Class/target CSV file with one discrete label per instance (e.g. target.csv)
- Expression matrix and target CSV for the myeloid progenitor dataset

## Outputs

- 2D embedding images per method (img_1_clean.pdf style exports for t-SNE, PCA, MDS, SPRING-style layouts, TSCAN pseudotime overlays)
- 2D_coords.txt coordinate files (one line per instance) and target.txt per embedding
- TSCAN pseudotime assignments over 6 (intestinal) and 20 (myeloid) clusters showing single-pathway detection and class splitting

## How to apply

Install GraphEmbed via PyPI after the pygraphviz dependency ('conda install pygraphviz'). Load the expression matrix and class/target CSV files for the intestinal epithelial and myeloid progenitor datasets. Run t-SNE with an elevated perplexity setting to reproduce the Supplementary Fig. 4 layout in which class separation is lost; compute PCA and MDS embeddings on the myeloid data to reproduce Supplementary Fig. 2. Build SPRING-style k-NN-only force layouts via graph_embed with -k 3 and -k 7 (class_confidence effectively 0), then rerun with -k 7 and a class-contraction prior via -c 1.5 to reproduce Supplementary Figs. 5-6. Run TSCAN with 6 clusters on the intestinal data and 20 clusters on the myeloid data, overlaying inferred pseudotime to reproduce the single-pathway detection and class-splitting patterns of Supplementary Figs. 7-8. Export each embedding as an image (e.g. img_1_clean.pdf) with its 2D_coords.txt file using the -o output directory option.

## Related tools

- **GraphEmbed** (Executed with -k only (class_confidence effectively 0) to produce SPRING-style k-NN force layouts, and with -c 1.5 to add the class-contraction prior for the baseline comparison) — github.com/fabriziocosta/GraphEmbed
- **TSCAN** (Pseudotime-based baseline: run with 6 clusters on intestinal and 20 clusters on myeloid data to test whether pseudotime alone detects the differentiation pathways) — zji90/TSCAN
- **SPRING** (Conceptual reference for k-NN-only force layouts reproduced via GraphEmbed without class contraction)
- **pygraphviz** (Non-pip dependency installed via conda before installing GraphEmbed from PyPI)

## Examples

```
./graph_embed.py -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 7 -o out_baseline
```

## Evaluation signals

- t-SNE at elevated perplexity loses class separation, matching the Supplementary Fig. 4 layout
- TSCAN on the myeloid data detects only a single linear pathway even when given 20 clusters, with large pseudotime differences within individual clusters and class splitting relative to the original labels
- k-NN-only force layouts (-k 3, -k 7 without -c) mix clusters, while the -c 1.5 run contracts same-class instances
- PCA and MDS embeddings of the myeloid data reproduce the Supplementary Fig. 2 layouts
- Each run exports the expected files (img_1_clean.pdf, 2D_coords.txt, target.txt, log) into the timestamped -o output directory; since no quantitative metric is reported, accept outputs by expert review of the qualitative patterns

## Limitations

- No quantitative performance metrics are reported to support the claimed superiority of GraphDDP over t-SNE, PCA, or MDS, so validation is qualitative (expert review)
- No software versions or parameter settings for the compared tools (TSCAN, Monocle2, SPRING) are given in the discussion
- No dataset accessions or named data resources are cited for the intestinal epithelial and myeloid progenitor datasets, so inputs must be sourced externally
- TSCAN depends on cluster count choice; given the same 20 classes as GraphDDP it still detects only a single linear pathway on the myeloid data

## Evidence

- [results] TSCAN detects only a single linear pathway on the myeloid data even when given the same number of classes as GraphDDP (20), showing large pseudotime differences within individual clusters: [evidence span withheld — anchor status: not_found]
- [methods] To set a desired separation strength use -c, to specify the number of neighbors use -k: "To set a desired separation strength use -c, to specify the number of neighbors use -k"
- [methods] GraphEmbed can be installed via PyPI: "GraphEmbed can be installed via PyPI"
- [readme] README installation and usage: dependency and separation parameters: "a non pip dependency is graphviz, you can install this e.g. via 
conda install pygraphviz"
- [readme] README output files listing: "2D_coords.txt       The 2D coordinates, one line per instance"
- [discussion] Comparison tools named in the discussion: "including pseudotime-based methods like TSCAN and Monocle2, as well as k-nearest neighbor approaches like SPRING"
