---
name: single-cell-population-separation-assessment
description: Use when a trained deep learning model (Keras model JSON + weight bin,
  e.g. tmpMod./tmpWeight.bin) has classified single-cell gene expression data (e.g.
  Zheng sorted, Pancreas, CellBench) and you need to visually assess whether learned
  representations separate cell populations.
when_to_use_negative:
- You need hidden outputs from a branch of a branching model — layerUMAP cannot capture
  these; capture them manually via Jupyter Notebook instead
- You want quantitative per-cell classification metrics rather than visual representation
  assessment — use the model's saved predictions (predicting.py) instead
- Input is raw sequence or expression data not yet passed through a trained model
  — no hidden-layer representations exist to visualize
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: layerUMAP
  role: Command-line tool included in autoBioSeqpy that projects last hidden-layer
    representations into two-dimensional UMAP space and supports grid search over
    n_neighbors/min_dist
  repo: github.com/jingry/autoBioSeqpy
- name: autoBioSeqpy
  role: Loads the trained Keras model, weights and training parameters and extracts
    hidden-layer representations of single-cell gene expression data
  repo: github.com/jingry/autoBioSeqpy
- name: UMAP
  role: Dimensionality-reduction library invoked by layerUMAP to embed representations
    in 2D under ten distance metrics
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2022.105530
    title: 'layerUMAP: A tool for visualizing and understanding deep learning models
      in biological sequence classification using UMAP'
  source_content_hash: sha256:3114961ad1134dd0d5ba6ee99ba5f66a20a2896eb56fed106e93ced200add622
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jingry/autoBioSeqpy
  tools:
  - UMAP
  - autoBioSeqpy
  - layerUMAP
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-population-separation-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualize the last hidden-layer representations learned by a deep learning cell-type classifier (e.g. RFCN) in two-dimensional UMAP space using layerUMAP, coloring cells by known cell-type labels, and assess how UMAP parameters (n_neighbors, min_dist) and distance metrics affect separation of cell populations.

## When to use

Use when a trained deep learning model (Keras model JSON + weight bin, e.g. tmpMod./tmpWeight.bin) has classified single-cell gene expression data (e.g. Zheng sorted, Pancreas, CellBench) and you need to visually assess whether learned representations separate cell populations.

## When NOT to use

- You need hidden outputs from a branch of a branching model — layerUMAP cannot capture these; capture them manually via Jupyter Notebook instead
- You want quantitative per-cell classification metrics rather than visual representation assessment — use the model's saved predictions (predicting.py) instead
- Input is raw sequence or expression data not yet passed through a trained model — no hidden-layer representations exist to visualize

## Inputs

- Trained Keras model file (e.g. tmpMod.) and weights (tmpWeight.bin) with recorded training parameters (tmpOut/parameters.txt)
- Single-cell gene expression dataset with known cell-type labels (e.g. Zheng sorted, Pancreas, CellBench)
- layerUMAP parameters: --n_neighbors, --min_dist, UMAP distance metric

## Outputs

- Two-dimensional UMAP scatter plots of hidden-layer representations colored by cell-type label (e.g. umap_metric_comparison.png)
- Grid of 126 UMAP embeddings spanning n_neighbors × min_dist combinations showing min_dist-dependent separation of cell populations
- Ten metric-comparison panels showing cluster separation for nine metrics and mixed populations for wminkowski

## How to apply

Load the trained Keras model and weights with autoBioSeqpy, then run the layerUMAP command-line tool (command-line command 2) to project the last hidden layer representations of single-cell gene expression data into two-dimensional UMAP space, coloring cells by their known cell-type labels. To probe parameter sensitivity, run the layerUMAP grid search (command-line command 4) over --n_neighbors and --min_dist, exhausting 126 parameter combinations; on the Zheng sorted dataset, min_dist has a significant impact on the separation of cell populations. To probe metric sensitivity, run the UMAP projection ten times, once per distance metric (chebyshev, braycurtis, canberra, correlation, cosine, euclidean, mahalanobis, manhattan, minkowski, wminkowski); only wminkowski is expected to yield mixed, non-separated cell populations. The rationale is that layerUMAP integrates autoBioSeqpy with the UMAP library to access learned high-level representations and can guide development of better models.

## Related tools

- **layerUMAP** (Command-line tool included in autoBioSeqpy that projects last hidden-layer representations into two-dimensional UMAP space and supports grid search over n_neighbors/min_dist) — github.com/jingry/autoBioSeqpy
- **autoBioSeqpy** (Loads the trained Keras model, weights and training parameters and extracts hidden-layer representations of single-cell gene expression data) — github.com/jingry/autoBioSeqpy
- **UMAP** (Dimensionality-reduction library invoked by layerUMAP to embed representations in 2D under ten distance metrics)

## Examples

```
python running.py --paraFile parameters.txt
```

## Evaluation signals

- The 126-combination grid over --n_neighbors and --min_dist completes and saved figures show min_dist-dependent change in cell-population separation on the Zheng sorted dataset
- The composite figure umap_metric_comparison.png shows nine of the ten distance metrics with clearly separated cell-population clusters and wminkowski alone failing to separate them
- Scatter plots are colored by the original publication's cell ontology labels, enabling visual inspection of cell population separation
- The tool runs in approximately 30 s per command, excluding interaction time (hardware-dependent)

## Limitations

- layerUMAP cannot capture the hidden outputs from the layers in a branch (if existing) of the built model; branch-layer outputs must be captured manually via Jupyter Notebook
- The actual text of command-line commands 1-5 is only referenced via Figure 2, not given inline
- Reported runtimes (approximately 30 s) are approximate and hardware-dependent without full specification

## Evidence
