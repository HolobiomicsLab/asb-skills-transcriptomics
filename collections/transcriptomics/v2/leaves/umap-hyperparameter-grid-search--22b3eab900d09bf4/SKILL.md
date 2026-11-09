---
name: umap-hyperparameter-grid-search
description: Use when 2D UMAP visualizations of deep-learning (e.g. RFCN) hidden-layer
  representations of scRNA-seq cells look unstable or poorly separated and you need
  to determine whether n_neighbors or min_dist drives the clustering before trusting
  the embedding.
when_to_use_negative:
- The model's hidden outputs live in a branch of the built network — layerUMAP cannot
  capture branched-layer outputs automatically; capture them manually via Jupyter
  Notebook instead.
- You need quantitative clustering metrics rather than visual separation assessment
  — the skill's validation criterion is the visual, min_dist-dependent change in cell-population
  separation, not a numeric score.
- The input is raw sequence/expression data rather than extracted hidden-layer representations
  — run autoBioSeqpy training/feature extraction first (running.py) before any UMAP
  visualization.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: layerUMAP
  role: Command-line tool (included in autoBioSeqpy) that runs the UMAP n_neighbors/min_dist
    grid search over hidden-layer representations
  repo: https://github.com/jingry/autoBioSeqpy
- name: autoBioSeqpy
  role: Loads the trained RFCN model and scRNA-seq data and extracts last hidden-layer
    representations fed to the UMAP grid search
  repo: https://github.com/jingry/autoBioSeqpy
- name: UMAP
  role: Dimensionality-reduction library whose n_neighbors and min_dist parameters
    are exhaustively swept (126 combinations)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2022.105530
    title: 'layerUMAP: A tool for visualizing and understanding deep learning models
      in biological sequence classification using UMAP'
  source_content_hash: sha256:fe479512ec6133d377774399c2c24934e313ee0a515a80cb2dd2ce562886237b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jingry/autoBioSeqpy
  repo_url: https://github.com/jingry/autoBioSeqpy
  tools:
  - UMAP
  - autoBioSeqpy
  - layerUMAP
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# umap-hyperparameter-grid-search

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A sensitivity-analysis skill that exhaustively sweeps UMAP embedding parameters (n_neighbors and min_dist) over 126 combinations using the layerUMAP grid-search command to assess how each parameter changes the visual separation of cell populations in learned RFCN representations, as demonstrated on the Zheng sorted scRNA-seq dataset.

## When to use

Use when 2D UMAP visualizations of deep-learning (e.g. RFCN) hidden-layer representations of scRNA-seq cells look unstable or poorly separated and you need to determine whether n_neighbors or min_dist drives the clustering before trusting the embedding.

## When NOT to use

- The model's hidden outputs live in a branch of the built network — layerUMAP cannot capture branched-layer outputs automatically; capture them manually via Jupyter Notebook instead.
- You need quantitative clustering metrics rather than visual separation assessment — the skill's validation criterion is the visual, min_dist-dependent change in cell-population separation, not a numeric score.
- The input is raw sequence/expression data rather than extracted hidden-layer representations — run autoBioSeqpy training/feature extraction first (running.py) before any UMAP visualization.

## Inputs

- Trained RFCN cell-type classification model (Keras model + weights saved by autoBioSeqpy, e.g. tmpMod. / tmpWeight.bin)
- scRNA-seq dataset with known cell-type labels (e.g. Zheng sorted dataset)
- Last hidden-layer representation matrix extracted from the trained model for all cells

## Outputs

- 126 UMAP 2D scatter plots, one per n_neighbors/min_dist combination, colored by cell-type labels
- Grid-search summary supporting the sensitivity conclusion (e.g. Figure S1 of the paper)
- Saved UMAP embedding outputs from layerUMAP (outSaveFolderPath-style artifacts)

## How to apply

First obtain last hidden-layer representations of all cells by loading the trained RFCN cell-type classification model together with the scRNA-seq data through autoBioSeqpy. Then run the layerUMAP grid search (command-line command 4) sweeping --n_neighbors and --min_dist to exhaust 126 UMAP parameter combinations, rendering a 2D UMAP scatter plot per combination. Color each scatter plot by the known cell-type labels so cluster separation can be judged visually. Compare plots across min_dist values: in the paper's Zheng sorted example, min_dist had a significant impact on cell-population separation while n_neighbors mattered less, so the decision point is which parameter's variation changes separation. Accept the run only if all 126 combinations complete and the figure set reproduces the min_dist-dependent change in separation reported in Figure 2 / Figure S1.

## Related tools

- **layerUMAP** (Command-line tool (included in autoBioSeqpy) that runs the UMAP n_neighbors/min_dist grid search over hidden-layer representations) — https://github.com/jingry/autoBioSeqpy
- **autoBioSeqpy** (Loads the trained RFCN model and scRNA-seq data and extracts last hidden-layer representations fed to the UMAP grid search) — https://github.com/jingry/autoBioSeqpy
- **UMAP** (Dimensionality-reduction library whose n_neighbors and min_dist parameters are exhaustively swept (126 combinations))

## Evaluation signals

- The grid search completes all 126 n_neighbors/min_dist parameter combinations without error.
- A 2D UMAP scatter plot is produced for every combination, with cells colored by their known cell-type labels.
- The saved figure set shows a visible change in cell-population separation as min_dist varies, matching the finding reported for the Zheng sorted dataset in Figure 2 / Figure S1.
- Representations used as UMAP input correspond to the last hidden layer of the trained RFCN model (verified via layerUMAP's layer-selection behavior / command-line command 2 baseline).
- Dependency environment resolves cleanly (numpy, matplotlib, sklearn, keras via anaconda3 or pip install -r requirements.txt) so layerUMAP runs reproducibly.

## Limitations

- layerUMAP cannot capture hidden outputs from layers in a branch of the built model; such outputs must be captured manually via Jupyter Notebook.
- The actual text of command-line command 4 (the grid-search invocation) is only referenced via Figure 2 and not given inline in the article, a reproducibility gap.
- Reported runtimes (e.g. ~30 s for related layerUMAP commands) are approximate and hardware-dependent without full hardware specification.
- The sensitivity conclusion (min_dist matters most) is demonstrated on the Zheng sorted dataset and may not transfer to other datasets or model architectures without re-running the grid.

## Evidence
