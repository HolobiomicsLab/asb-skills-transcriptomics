---
name: command-line-bioinformatics-workflow
description: Use when a trained Keras model for protein/DNA/RNA sequences or single-cell
  gene expression (e.g. FASTA test files or Zheng sorted data) must be re-run, applied
  for prediction, or dissected layer-by-layer in a noncoding, unified command-line
  format.
when_to_use_negative:
- The goal is to capture hidden outputs from layers inside a branch of a multi-branch
  built model — layerUMAP cannot do this; use the related Jupyter Notebook manually
  instead
- An interactive, coding-based analysis is preferred over the unified command-line
  interface, e.g. when a custom UMAP or t-SNE configuration is needed beyond the tool's
  parameters
- The data are not biological sequences or single-cell expression matrices encodable
  by autoBioSeqpy (protein/DNA/RNA dict or other supported encodings with a defined
  spcLen)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: autoBioSeqpy
  role: Standalone command-line framework (running.py, predicting.py) that encodes
    biological sequences into matrices and trains/applies the Keras DL models whose
    outputs layerUMAP visualizes
  repo: https://github.com/jingry/autoBioSeqpy
- name: layerUMAP
  role: Command-line tool included in autoBioSeqpy that integrates the UMAP library
    to access and visualize hidden-layer representations of trained models layer by
    layer
  repo: https://github.com/jingry/autoBioSeqpy
- name: UMAP
  role: Library used by layerUMAP to project last-hidden-layer representations into
    2D, with grid-searchable n_neighbors and min_dist parameters and per-metric distance
    settings
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2022.105530
    title: 'layerUMAP: A tool for visualizing and understanding deep learning models
      in biological sequence classification using UMAP'
  source_content_hash: sha256:6960394f49d6569075aacb4d738bb4c10bb1af71100730f8a1eddc1fbd2fb54a
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
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# command-line-bioinformatics-workflow

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs deep learning model training, prediction, and hidden-layer visualization for biological sequence and single-cell data as standalone command-line applications (autoBioSeqpy running.py/predicting.py plus layerUMAP), so that model development and UMAP-based interpretation proceed without custom coding.

## When to use

Use when a trained Keras model for protein/DNA/RNA sequences or single-cell gene expression (e.g. FASTA test files or Zheng sorted data) must be re-run, applied for prediction, or dissected layer-by-layer in a noncoding, unified command-line format.

## When NOT to use

- The goal is to capture hidden outputs from layers inside a branch of a multi-branch built model — layerUMAP cannot do this; use the related Jupyter Notebook manually instead
- An interactive, coding-based analysis is preferred over the unified command-line interface, e.g. when a custom UMAP or t-SNE configuration is needed beyond the tool's parameters
- The data are not biological sequences or single-cell expression matrices encodable by autoBioSeqpy (protein/DNA/RNA dict or other supported encodings with a defined spcLen)

## Inputs

- FASTA protein sequence files split into positive/negative sets (e.g. examples/typeIII_secreted_effectors_prediction/data/train_pos.txt, train_neg.txt, test_pos.txt, test_neg.txt)
- A model architecture loader file (e.g. examples/typeIII_secreted_effectors_prediction/model/protein_CNN_model.py or protein_biLSTM_model.py)
- A saved Keras model (tmpMod.) with weights (tmpWeight.bin) and recorded training parameters (parameters.txt) for prediction or hidden-layer visualization
- Single-cell gene expression data (e.g. Zheng sorted dataset) with cell ontology labels for RFCN visualization
- A consolidated parameter file passed via --paraFile

## Outputs

- Trained Keras model file tmpMod. with weights tmpWeight.bin and training record parameters.txt saved to --outSaveFolderPath tmpOut
- Saved predictions (e.g. indPredictions.txt via --predictionSavePath) and figures (--saveFig 1)
- 2D UMAP scatter plots of last-hidden-layer representations colored by class label (e.g. umap_last_hidden_layer.png, umap_metric_comparison.png)
- Layer-by-layer visualization of model hidden activities along model depth

## How to apply

Install dependencies (re, numpy, importlib, matplotlib, sklearn, keras — bundled with anaconda3) and either run from the autoBioSeqpy root directory or append the folder to sys.path via sys.path.append(libPath). Train models with running.py, passing parameters such as --dataType protein --dataEncodingType dict --spcLen 100 --batch_size 60 --epochs 20, saving the Keras model as tmpMod., weights as tmpWeight.bin, and parameters as parameters.txt; alternatively consolidate all flags into parameters.txt and invoke via --paraFile. Re-apply trained models with predicting.py using --paraFile tmpOut/parameters.txt plus the test file paths, which reuses the recorded encoding so only the test FASTA paths and optional --predictionSavePath are needed. For interpretation, use the layerUMAP command-line tool (commands 2-5) to visualize last hidden layer representations in 2D UMAP space, grid-searching UMAP parameters n_neighbors and min_dist (min_dist notably affects cell-population separation on the Zheng sorted dataset) and testing distance metrics such as chebyshev, braycurtis, canberra, correlation, cosine, euclidean, mahalanatallan, manhattan, minkowski, and wminkowski. Enable --saveFig 1 --outSaveFolderPath tmpOut so figures and predictions land in the designated output folder for validation.

## Related tools

- **autoBioSeqpy** (Standalone command-line framework (running.py, predicting.py) that encodes biological sequences into matrices and trains/applies the Keras DL models whose outputs layerUMAP visualizes) — https://github.com/jingry/autoBioSeqpy
- **layerUMAP** (Command-line tool included in autoBioSeqpy that integrates the UMAP library to access and visualize hidden-layer representations of trained models layer by layer) — https://github.com/jingry/autoBioSeqpy
- **UMAP** (Library used by layerUMAP to project last-hidden-layer representations into 2D, with grid-searchable n_neighbors and min_dist parameters and per-metric distance settings)

## Examples

```
python running.py --paraFile parameters.txt
```

## Evaluation signals

- python running.py --help (or --paraFile) runs without error after dependency resolution, confirming the standalone application is available
- Output folder contains the expected artifacts: tmpMod., tmpWeight.bin, parameters.txt, saved predictions, and figures when --savePrediction 1 --saveFig 1 are set
- predicting.py reproduces the training-time encoding automatically from tmpOut/parameters.txt, so the same test FASTA files yield consistent per-sequence predictions
- The UMAP figure (e.g. umap_last_hidden_layer.png) shows distinguishable clusters by true class label, consistent with the reported classification accuracy (e.g. nine of ten UMAP distance metrics separate cell populations while wminkowski alone mixes them)
- The UMAP parameter grid search demonstrates the known sensitivity of cluster separation to min_dist on the Zheng sorted dataset

## Limitations

- layerUMAP currently cannot capture hidden outputs from layers located in a branch (if existing) of the built model; branch outputs must be captured manually using the related Jupyter Notebook
- The actual text of the layerUMAP command-line commands is only referenced via a figure in the paper rather than given inline, creating a reproducibility gap
- Reported runtimes (approximately 30 s) are approximate and hardware-dependent without full specification
- All code is pure Python relying on keras and sklearn; module dependencies must be resolved first (anaconda3 recommended, Python 2 not suggested)

## Evidence

- [intro] We present layerUMAP, a command-line tool included in the autoBioSeqpy software, written in Python, which makes use of UMAP to understand, diagnose and refine DL models.: "We present layerUMAP, a command-line tool included in the autoBioSeqpy software, written in Python, which makes use of UMAP to understand, diagnose and refine DL models."
- [discussion] Currently, layerUMAP cannot capture the hidden outputs from the layers in a branch (if existing) of the built model: "Currently, layerUMAP cannot capture the hidden outputs from the layers in a branch (if existing) of the built model"
- [readme] python running.py --paraFile parameters.txt: "This file can then be used in a command line: `python running.py --paraFile parameters.txt`"
- [methods] users could resolve module dependencies by installing anaconda3 (2 is not suggested) on their official website: "users could resolve module dependencies by installing anaconda3 (2 is not suggested) on their official website"
- [methods] --dataTrainFilePaths examples/typeIII_secreted_effectors_prediction/data/train_pos.txt examples/typeIII_secreted_effectors_prediction/data/train_neg.txt: "--dataTrainFilePaths examples/typeIII_secreted_effectors_prediction/data/train_pos.txt examples/typeIII_secreted_effectors_prediction/data/train_neg.txt"
