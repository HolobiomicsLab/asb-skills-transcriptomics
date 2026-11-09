---
name: model-architecture-layer-dissection
description: Use when a trained Keras model (model JSON, weights, parameters.txt from
  an autoBioSeqpy run) must be inspected layer by layer to understand learned representations,
  e.g. dissecting a protein-CNN on typeIII secreted effector sequences or an RFCN
  on single-cell data.
when_to_use_negative:
- The hidden layer of interest lies in a branch of the built model — layerUMAP cannot
  capture branch-layer outputs; users must capture them manually in a Jupyter Notebook
- The goal is final classification accuracy or hyperparameter selection for training
  (e.g. grid search over filter length and number of filters) rather than inspecting
  learned hidden representations
- The model was not trained/exported with autoBioSeqpy-compatible Keras artifacts
  (model JSON + weights + parameters.txt)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: layerUMAP
  role: Command-line tool that builds a multi-output view of the trained Keras model,
    exposes hidden-layer activations along model depth with interactive layer selection,
    and visualizes them with UMAP
  repo: github.com/jingry/autoBioSeqpy
- name: autoBioSeqpy
  role: Trains/exports the Keras model artifacts and encodes biological sequences
    into matrices (dict encoding, --spcLen 100) that layerUMAP consumes
  repo: github.com/jingry/autoBioSeqpy
- name: UMAP
  role: Library used to embed each layer's reduced feature matrix into a 2-D space
    for the layer-wise scatter plots
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2022.105530
    title: 'layerUMAP: A tool for visualizing and understanding deep learning models
      in biological sequence classification using UMAP'
  source_content_hash: sha256:cc557ac4772c58f8632f77f8b86c73ef837441e3719b3fd4d38f97ba6976ffa7
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# model-architecture-layer-dissection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Layer-by-layer dissection of trained autoBioSeqpy deep learning models using layerUMAP, which exposes hidden-layer activations along model depth and embeds each layer's representation in 2-D UMAP space to diagnose how CNN, BiLSTM/BiGRU or RFCN architectures separate biological sequence classes.

## When to use

Use when a trained Keras model (model JSON, weights, parameters.txt from an autoBioSeqpy run) must be inspected layer by layer to understand learned representations, e.g. dissecting a protein-CNN on typeIII secreted effector sequences or an RFCN on single-cell data.

## When NOT to use

- The hidden layer of interest lies in a branch of the built model — layerUMAP cannot capture branch-layer outputs; users must capture them manually in a Jupyter Notebook
- The goal is final classification accuracy or hyperparameter selection for training (e.g. grid search over filter length and number of filters) rather than inspecting learned hidden representations
- The model was not trained/exported with autoBioSeqpy-compatible Keras artifacts (model JSON + weights + parameters.txt)

## Inputs

- Trained Keras model JSON (e.g. tmpMod.json from --modelSaveName)
- Trained model weights (e.g. tmpWeight.bin from --weightSaveName)
- Training parameters file (parameters.txt saved under tmpOut via --paraSaveName)
- Encoded biological sequence data (protein FASTA/txt sequences encoded with dict encoding and --spcLen 100)
- Class labels for train/test samples

## Outputs

- Per-layer reduced 2-D sample-by-feature matrices of hidden-layer activations
- 2-D UMAP embeddings of each selected main-path hidden layer
- Layer-wise scatter plots colored by class label (layer-by-layer dissection figures)

## How to apply

Load the trained Keras artifacts produced by running.py (--modelSaveName tmpMod.json, --weightSaveName tmpWeight.bin, and the recorded training settings in parameters.txt under tmpOut), then re-encode the input sequences with the autoBioSeqpy data-processing modules using dict encoding and --spcLen 100. Traverse model.layers along the main path to build a multi-output Keras model returning every hidden layer's activation; note that branch layers cannot be captured. For outputs with rank greater than 2, reduce them to a 2-D sample-by-feature matrix via 'max', 'min' or 'avg' pooling over the channel axis, or 'flatten'. In --interactive mode, select a layer from the listed layer names/indices along model depth, embed the reduced matrix with the UMAP library, and render a scatter plot colored by class label; a grid search over UMAP parameters n_neighbors and min_dist can tune cell-population separation (min_dist showed significant impact on the Zheng sorted dataset).

## Related tools

- **layerUMAP** (Command-line tool that builds a multi-output view of the trained Keras model, exposes hidden-layer activations along model depth with interactive layer selection, and visualizes them with UMAP) — github.com/jingry/autoBioSeqpy
- **autoBioSeqpy** (Trains/exports the Keras model artifacts and encodes biological sequences into matrices (dict encoding, --spcLen 100) that layerUMAP consumes) — github.com/jingry/autoBioSeqpy
- **UMAP** (Library used to embed each layer's reduced feature matrix into a 2-D space for the layer-wise scatter plots)

## Examples

```
python running.py --dataType protein --dataEncodingType dict --spcLen 100 --modelSaveName tmpMod.json --weightSaveName tmpWeight.bin --paraSaveName parameters.txt
```

## Evaluation signals

- The module runs end-to-end on the shipped example CNN model from examples/typeIII_secreted_effectors_prediction
- Every main-path layer yields a valid 2-D UMAP embedding and scatter plot colored by class label
- The interactive mode lists layer names and indices along model depth for user selection
- Reduction of rank>2 outputs to a 2-D sample-by-feature matrix succeeds via max/min/avg pooling or flatten
- The tool exposes no API for capturing branch-layer outputs, matching the documented supported behavior

## Limitations

- layerUMAP cannot capture the hidden outputs from layers in a branch of the built model; these must be captured manually via a Jupyter Notebook
- The actual text of command-line commands 1-5 (including the layer-by-layer dissection command) is only shown in Figure 2, creating a reproducibility gap
- Reported runtimes (~30 s excluding interaction time) are approximate and hardware-dependent without full specification
- UMAP parameters matter: min_dist had a significant impact on the separation of cell populations (Zheng sorted dataset example)

## Evidence
