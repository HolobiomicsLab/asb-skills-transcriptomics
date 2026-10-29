---
name: deep-learning-model-representation-visualization
description: Use when a trained Keras deep learning classifier (e.g. RFCN on scRNA-seq
  data such as the Zheng sorted dataset) must be inspected or diagnosed by visualizing
  its hidden-layer outputs, or when the sensitivity of the visualization to UMAP parameters
  (n_neighbors, min_dist) must be assessed.
when_to_use_negative:
- 'The model contains branch layers: layerUMAP cannot capture hidden outputs from
  layers in a branch, so outputs must be captured manually in a Jupyter Notebook instead'
- The goal is quantitative classification accuracy (e.g. RFCN's 98.09% on Zheng sorted)
  rather than visual inspection of learned representations
- 'The exact command-line text is required but unavailable: the article only references
  commands 1-5 via Figure 2, creating a reproducibility gap'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: layerUMAP
  role: Command-line tool (inside autoBioSeqpy) that extracts hidden-layer representations
    from the trained model and embeds them with UMAP; used to run the n_neighbors/min_dist
    grid search (command 4)
  repo: github.com/jingry/autoBioSeqpy
- name: autoBioSeqpy
  role: Host software that loads the trained model and encodes scRNA-seq input data,
    supplying the hidden-layer activations that layerUMAP visualizes
  repo: github.com/jingry/autoBioSeqpy
- name: UMAP
  role: Dimensionality-reduction library invoked by layerUMAP to project representations
    into 2D; its n_neighbors and min_dist parameters are grid-searched over 126 combinations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2022.105530
    title: 'layerUMAP: A tool for visualizing and understanding deep learning models
      in biological sequence classification using UMAP'
  source_content_hash: sha256:c71be613f236fb41742376f7050eba46ba73f35c740fa9c3a1a456cc79b7083c
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
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# deep-learning-model-representation-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A skill for inspecting the high-level representations learned by a trained deep learning model — here the RFCN cell-type classifier — by projecting the last hidden layer activations into two-dimensional UMAP space with layerUMAP, including a grid search over UMAP's n_neighbors and min_dist parameters.

## When to use

Use when a trained Keras deep learning classifier (e.g. RFCN on scRNA-seq data such as the Zheng sorted dataset) must be inspected or diagnosed by visualizing its hidden-layer outputs, or when the sensitivity of the visualization to UMAP parameters (n_neighbors, min_dist) must be assessed.

## When NOT to use

- The model contains branch layers: layerUMAP cannot capture hidden outputs from layers in a branch, so outputs must be captured manually in a Jupyter Notebook instead
- The goal is quantitative classification accuracy (e.g. RFCN's 98.09% on Zheng sorted) rather than visual inspection of learned representations
- The exact command-line text is required but unavailable: the article only references commands 1-5 via Figure 2, creating a reproducibility gap

## Inputs

- Trained deep learning model files (keras model JSON + weights, e.g. modelSaveName/tmpMod. and tmpWeight.bin from autoBioSeqpy)
- scRNA-seq gene expression data (e.g. Zheng sorted dataset) loadable by autoBioSeqpy data readers
- Known cell-type labels for each cell (used to color UMAP scatter points)
- autoBioSeqpy installation with layerUMAP and the UMAP library, with module paths set via sys.path.append

## Outputs

- Two-dimensional UMAP scatter plots of last hidden layer representations, colored by cell-type labels
- A grid of 126 UMAP embeddings spanning n_neighbors/min_dist combinations (Figure S1-style sensitivity analysis)
- Interactive layer-by-layer visualization of hidden-layer outputs along model depth

## How to apply

Load the trained model and data with autoBioSeqpy and extract the last hidden layer representations for all cells; layerUMAP (a command-line tool included in autoBioSeqpy) then embeds these representations into two-dimensional UMAP space. For the Zheng sorted dataset, run the layerUMAP grid search (command-line command 4 in the article's Figure 2) over --n_neighbors and --min_dist, exhausting 126 parameter combinations, and render a 2D UMAP scatter per combination with cells colored by known cell-type labels. Compare cluster separation across min_dist values: the article reports that min_dist had a significant impact on separation of cell populations on Zheng sorted, so the saved figure grid should show visible min_dist-dependent changes. layerUMAP also offers an interactive option to visualize hidden-layer outputs along model depth (e.g. layer-by-layer CNN dissection with command 3). Dependencies (numpy, matplotlib, sklearn, keras) are resolved via anaconda3 or pip, and the autoBioSeqpy folder is added to sys.path if used as a module.

## Related tools

- **layerUMAP** (Command-line tool (inside autoBioSeqpy) that extracts hidden-layer representations from the trained model and embeds them with UMAP; used to run the n_neighbors/min_dist grid search (command 4)) — github.com/jingry/autoBioSeqpy
- **autoBioSeqpy** (Host software that loads the trained model and encodes scRNA-seq input data, supplying the hidden-layer activations that layerUMAP visualizes) — github.com/jingry/autoBioSeqpy
- **UMAP** (Dimensionality-reduction library invoked by layerUMAP to project representations into 2D; its n_neighbors and min_dist parameters are grid-searched over 126 combinations)

## Evaluation signals

- The UMAP parameter grid completes all 126 n_neighbors/min_dist combinations without error
- The saved figure grid shows min_dist-dependent changes in cell-population separation on the Zheng sorted dataset, matching the reported finding
- Cluster separation in the 2D UMAP scatter is consistent with RFCN's reported 98.09% overall accuracy on Zheng sorted cells
- Each scatter plot is correctly colored by known cell-type labels rather than predicted labels when reproducing the sensitivity analysis
- The interactive layer-depth view (command 3) yields progressively more separated cell populations toward the last hidden layer

## Limitations

- layerUMAP cannot capture hidden outputs from layers in a branch of the built model; branch outputs must be captured manually via Jupyter Notebook
- The actual text of command-line commands 1-5 is only referenced via Figure 2, not given inline, complicating exact reproduction
- Runtimes are approximate and hardware-dependent (e.g. ~30 s reported for one command, excluding interaction time) without full hardware specification

## Evidence
