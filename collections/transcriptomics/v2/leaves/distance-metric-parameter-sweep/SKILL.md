---
name: distance-metric-parameter-sweep
description: Use when you have hidden-layer embeddings from a trained Keras model
  (e.g. RFCN on single-cell gene expression) and need to check that two-dimensional
  UMAP cell-type separation does not depend on the choice of distance metric before
  reporting the visualization.
when_to_use_negative:
- The goal is to sweep model hyperparameters (convolution layers, filter length, number
  of filters, BiLSTM/BiGRU nodes) rather than the UMAP distance metric of an already
  trained model
- The input is raw sequence or expression data rather than hidden-layer representations
  extracted from a trained Keras model loaded via autoBioSeqpy
- Only a single final UMAP figure is needed and metric sensitivity has already been
  established for the same model/dataset
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: layerUMAP
  role: Command-line tool (included in autoBioSeqpy) that applies UMAP to hidden layers
    of the trained model; the metric parameter is varied across the sweep
  repo: github.com/jingry/autoBioSeqpy
- name: autoBioSeqpy
  role: Loads the trained Keras model, weights, and parameters and extracts hidden-layer
    outputs of the single-cell expression data
  repo: github.com/jingry/autoBioSeqpy
- name: UMAP
  role: Dimensionality-reduction library underlying layerUMAP; accepts the metric
    parameter (chebyshev, braycurtis, canberra, correlation, cosine, euclidean, mahalanobis,
    manhattan, minkowski, wminkowski)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2022.105530
    title: 'layerUMAP: A tool for visualizing and understanding deep learning models
      in biological sequence classification using UMAP'
  source_content_hash: sha256:3e07eda9954a05b6d1ce6a57f43dd514a4e9c83b571cc2d9b076b12186ac1a31
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# distance-metric-parameter-sweep

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A sensitivity sweep in which a UMAP projection of deep-learned hidden-layer representations (e.g. RFCN outputs on the Zheng sorted single-cell dataset) is repeated once per distance metric to test whether visual cluster separation of cell types is robust to the metric choice.

## When to use

Use when you have hidden-layer embeddings from a trained Keras model (e.g. RFCN on single-cell gene expression) and need to check that two-dimensional UMAP cell-type separation does not depend on the choice of distance metric before reporting the visualization.

## When NOT to use

- The goal is to sweep model hyperparameters (convolution layers, filter length, number of filters, BiLSTM/BiGRU nodes) rather than the UMAP distance metric of an already trained model
- The input is raw sequence or expression data rather than hidden-layer representations extracted from a trained Keras model loaded via autoBioSeqpy
- Only a single final UMAP figure is needed and metric sensitivity has already been established for the same model/dataset

## Inputs

- Trained Keras model file (tmpMod.) and weights (tmpWeight.bin) from autoBioSeqpy training
- Training parameters file (tmpOut/parameters.txt)
- Single-cell gene expression data from the Zheng sorted dataset (cell type labels from the original publication)

## Outputs

- Ten two-dimensional UMAP scatter plots of last hidden-layer representations, one per distance metric, colored by cell ontology label
- Composite figure (umap_metric_comparison.png) contrasting the nine well-separated metrics with the mixed wminkowski result

## How to apply

Load the trained model (tmpMod.), its weights (tmpWeight.bin) and the recorded training parameters (tmpOut/parameters.txt) via autoBioSeqpy, then use the layerUMAP command-line tool, which integrates autoBioSeqpy with the UMAP library, to project the last hidden-layer representations of the single-cell expression data. Run the projection ten times, setting the metric parameter to chebyshev, braycurtis, canberra, correlation, cosine, euclidean, mahalanobis, manhattan, minkowski, and wminkowski in turn. Render each result as a two-dimensional UMAP scatter plot colored by cell ontology label taken from the original publication. The decision point is the cluster-separation pattern: nine of the ten metrics should yield visually distinct cell-population clusters, and a finding that only wminkowski produces mixed, non-separated populations confirms the reported layerUMAP behavior. If separation varies with other UMAP settings instead, note that the authors observed the parameter 'min_dist' had a significant impact on separation and sweep it via the grid-search command as a follow-up.

## Related tools

- **layerUMAP** (Command-line tool (included in autoBioSeqpy) that applies UMAP to hidden layers of the trained model; the metric parameter is varied across the sweep) — github.com/jingry/autoBioSeqpy
- **autoBioSeqpy** (Loads the trained Keras model, weights, and parameters and extracts hidden-layer outputs of the single-cell expression data) — github.com/jingry/autoBioSeqpy
- **UMAP** (Dimensionality-reduction library underlying layerUMAP; accepts the metric parameter (chebyshev, braycurtis, canberra, correlation, cosine, euclidean, mahalanobis, manhattan, minkowski, wminkowski))

## Examples

```
for m in chebyshev braycurtis canberra correlation cosine euclidean mahalanobis manhattan minkowski wminkowski; do python layerUMAP.py --paraFile tmpOut/parameters.txt --metric $m; done
```

## Evaluation signals

- Exactly ten UMAP runs are produced, one per metric: chebyshev, braycurtis, canberra, correlation, cosine, euclidean, mahalanobis, manhattan, minkowski, wminkowski
- Nine of the ten plots show visually distinct, well-separated cell-population clusters when colored by cell ontology label
- The wminkowski run alone yields mixed, non-separated cell populations, matching the reported layerUMAP behavior
- The composite umap_metric_comparison.png assembles all ten panels and is consistent with the figure described in the source card
- The projection uses the last hidden-layer representations of the RFCN model, not the raw expression input

## Limitations

- layerUMAP cannot capture hidden outputs from layers inside a branch of the built model; branch-layer outputs must be captured manually in a Jupyter Notebook
- The actual text of the layerUMAP command-line commands is only shown in Figure 2 of the article, not inline, creating a reproducibility gap
- Reported runtimes (~30 s per command) are approximate and hardware-dependent without full specification; UMAP parameters such as min_dist and n_neighbors strongly affect separation and may need a separate grid search

## Evidence
