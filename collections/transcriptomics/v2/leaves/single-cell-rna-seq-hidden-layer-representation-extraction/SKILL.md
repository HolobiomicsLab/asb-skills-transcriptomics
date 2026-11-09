---
name: single-cell-rna-seq-hidden-layer-representation-extraction
description: Use when a trained DL model (e.g., RFCN on the Zheng sorted scRNA-seq
  dataset) exists and you need its last hidden layer outputs projected into a 2D UMAP
  space to inspect cell-population separation by known cell-type labels.
when_to_use_negative:
- 'The model architecture contains branched layers: ''Currently, layerUMAP cannot
  capture the hidden outputs from the layers in a branch (if existing) of the built
  model'', so branch-layer outputs must instead be captured manually via Jupyter Notebook.'
- You only have raw sequences or raw expression data and no trained DL model — use
  autoBioSeqpy's running.py training workflow first rather than layerUMAP.
- You need a noncoding, unified-format tool for something other than visualizing hidden-layer
  activities (layerUMAP is specific to model-introspection via UMAP).
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: layerUMAP
  role: Command-line tool (included in autoBioSeqpy) that runs UMAP on the extracted
    hidden layer representations and provides the --n_neighbors/--min_dist grid search
    and interactive layer-by-layer visualization
  repo: github.com/jingry/autoBioSeqpy
- name: autoBioSeqpy
  role: Loads the trained RFCN model and single-cell gene expression data and exposes
    the last hidden layer outputs for visualization
  repo: github.com/jingry/autoBioSeqpy
- name: UMAP
  role: Dimensionality-reduction library integrated by layerUMAP to embed hidden layer
    representations in 2D
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2022.105530
    title: 'layerUMAP: A tool for visualizing and understanding deep learning models
      in biological sequence classification using UMAP'
  source_content_hash: sha256:2a2ef225ffcd0e511a7219a6ed4fb59ab08c44116932be22d22cf4ba496416ce
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# single-cell-rna-seq-hidden-layer-representation-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts the last hidden layer representations of a trained deep learning classifier (e.g., the RFCN model) on single-cell gene expression data using autoBioSeqpy, then embeds them in 2D with layerUMAP/UMAP. This makes learned high-level cell-type representations visually inspectable and enables parameter-sensitivity analysis of the embedding.

## When to use

Use when a trained DL model (e.g., RFCN on the Zheng sorted scRNA-seq dataset) exists and you need its last hidden layer outputs projected into a 2D UMAP space to inspect cell-population separation by known cell-type labels.

## When NOT to use

- The model architecture contains branched layers: 'Currently, layerUMAP cannot capture the hidden outputs from the layers in a branch (if existing) of the built model', so branch-layer outputs must instead be captured manually via Jupyter Notebook.
- You only have raw sequences or raw expression data and no trained DL model — use autoBioSeqpy's running.py training workflow first rather than layerUMAP.
- You need a noncoding, unified-format tool for something other than visualizing hidden-layer activities (layerUMAP is specific to model-introspection via UMAP).

## Inputs

- Trained deep learning classification model (e.g., RFCN saved as keras model + weights from autoBioSeqpy)
- Single-cell gene expression dataset with known cell-type labels (e.g., Zheng sorted scRNA-seq data)
- UMAP parameter grid for --n_neighbors and --min_dist (126 combinations as in the paper)

## Outputs

- Matrix of last hidden layer representations for all cells
- Two-dimensional UMAP scatter plots colored by cell-type labels, one per parameter combination
- Grid-search result showing the effect of n_neighbors/min_dist on cell-population separation (Figure S1-style output)

## How to apply

First install the Python dependencies (re, numpy, importlib, matplotlib, sklearn, keras), ideally via anaconda3 (Python 2 is not suggested), and add the autoBioSeqpy folder to sys.path or run from its root directory. Load the trained RFCN model and the single-cell gene expression data (e.g., the Zheng sorted dataset) with autoBioSeqpy and extract the last hidden layer representations for all cells, leveraging layerUMAP's ability to visualize hidden layer outputs along the depth of the model. Project these representations into 2D using layerUMAP, which integrates autoBioSeqpy with the UMAP library; for sensitivity analysis, run the layerUMAP grid search (command-line command 4) over --n_neighbors and --min_dist, exhausting 126 parameter combinations. Color each 2D scatter by known cell-type labels and compare cluster separation across parameter values; on the Zheng sorted dataset, min_dist has a significant impact on cell-population separation. Decide which embedding best reflects the model's learned cell-type structure based on visual separation of the labeled populations.

## Related tools

- **layerUMAP** (Command-line tool (included in autoBioSeqpy) that runs UMAP on the extracted hidden layer representations and provides the --n_neighbors/--min_dist grid search and interactive layer-by-layer visualization) — github.com/jingry/autoBioSeqpy
- **autoBioSeqpy** (Loads the trained RFCN model and single-cell gene expression data and exposes the last hidden layer outputs for visualization) — github.com/jingry/autoBioSeqpy
- **UMAP** (Dimensionality-reduction library integrated by layerUMAP to embed hidden layer representations in 2D)

## Evaluation signals

- The 126-combination UMAP grid search (--n_neighbors x --min_dist) completes without error and saves one figure per combination.
- The saved figures show the min_dist-dependent change in cell-population separation reported for the Zheng sorted dataset (Figure 2 / Figure S1).
- The extracted representations come from the last hidden layer of the RFCN model, consistent with layerUMAP's interactive depth-wise layer selection.
- Model quality baseline holds: RFCN achieved 98.09% overall accuracy on the Zheng sorted dataset, so cell populations should be separable in the embedding.
- Dependencies resolve cleanly per the README (anaconda3 or pip install -r requirements.txt) and running.py --help executes without error before layerUMAP runs.

## Limitations

- layerUMAP cannot capture hidden outputs from layers in a branch of the built model; users must capture branch-layer outputs manually using the related Jupyter Notebook.
- The actual text of command-line commands 1-5 is only referenced via Figure 2, not given inline in the article (reproducibility gap).
- Reported runtimes (approximately 30 s for a command, excluding interaction time) are approximate and hardware-dependent.
- UMAP embeddings are parameter-sensitive: on the Zheng sorted dataset, min_dist significantly affected cell-population separation, so results are not parameter-invariant.

## Evidence
