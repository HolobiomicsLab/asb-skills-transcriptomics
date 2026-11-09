---
name: tensor-dimensionality-reduction-pooling
description: Use when a trained Keras model's hidden-layer outputs have rank > 2 (e.g.,
  CNN feature maps from dict-encoded protein sequences) and you need a 2-D UMAP embedding
  to inspect representations layer by layer.
when_to_use_negative:
- The layer of interest lies in a branch of the built model — layerUMAP cannot capture
  hidden outputs from branch layers; capture them manually in a Jupyter Notebook instead.
- The hidden-layer output tensor already has rank ≤ 2 (e.g., a dense layer output),
  where no pooling/flatten reduction step is needed before UMAP.
- The task requires model training or prediction rather than hidden-representation
  visualization — use autoBioSeqpy's running.py or predicting.py pipelines directly
  instead.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: layerUMAP
  role: Command-line tool that exposes hidden-layer outputs along model depth, applies
    pooling/flatten reduction to rank>2 tensors, and renders UMAP layer-wise visualizations
    with an interactive layer-selection mode
  repo: https://github.com/jingry/autoBioSeqpy
- name: autoBioSeqpy
  role: Provides the data-processing modules to encode biological sequences (dict
    encoding, spcLen 100) and saves the Keras model JSON/weights/parameters artifacts
    consumed by layerUMAP
  repo: https://github.com/jingry/autoBioSeqpy
- name: UMAP
  role: Library used by layerUMAP to embed the reduced 2-D hidden-layer feature matrices
    into two-dimensional space for visualization
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2022.105530
    title: 'layerUMAP: A tool for visualizing and understanding deep learning models
      in biological sequence classification using UMAP'
  source_content_hash: sha256:19196d230b00314bf6a123b10a874c7b2b4d4e0e49a208ad11f9db7f20a91300
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# tensor-dimensionality-reduction-pooling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reduce high-rank hidden-layer activation tensors from trained autoBioSeqpy/Keras models to a 2-D sample-by-feature matrix (via max/min/avg pooling or flatten), then embed with UMAP for layer-wise visualization of learned biological-sequence representations.

## When to use

Use when a trained Keras model's hidden-layer outputs have rank > 2 (e.g., CNN feature maps from dict-encoded protein sequences) and you need a 2-D UMAP embedding to inspect representations layer by layer.

## When NOT to use

- The layer of interest lies in a branch of the built model — layerUMAP cannot capture hidden outputs from branch layers; capture them manually in a Jupyter Notebook instead.
- The hidden-layer output tensor already has rank ≤ 2 (e.g., a dense layer output), where no pooling/flatten reduction step is needed before UMAP.
- The task requires model training or prediction rather than hidden-representation visualization — use autoBioSeqpy's running.py or predicting.py pipelines directly instead.

## Inputs

- Trained Keras model JSON (from --modelSaveName) and weights file (from --weightSaveName) produced by autoBioSeqpy
- Recorded training settings in parameters.txt under tmpOut
- Protein FASTA sequences encoded with autoBioSeqpy dict encoding at sequence length 100 (--spcLen 100)

## Outputs

- 2-D sample-by-feature matrix for each selected main-path hidden layer (after pooling or flatten reduction)
- 2-D UMAP embedding of the selected layer's representations
- Layer-wise scatter plot colored by class label

## How to apply

Load the trained Keras model artifacts saved by an autoBioSeqpy run (--modelSaveName tmpMod., --weightSaveName tmpWeight.bin, and parameters.txt in tmpOut), and encode the protein FASTA inputs with the autoBioSeqpy data-processing modules using dict encoding and --spcLen 100. Build a multi-output model by traversing model.layers along the main path so every hidden-layer activation is returned, skipping branch layers. For each output tensor of rank greater than 2, apply one of the user-selected reduction options — 'max', 'min', or 'avg' pooling over the channel/feature axis, or 'flatten' — to yield a 2-D sample-by-feature matrix. Embed the reduced matrix into two dimensions with the UMAP library and render a scatter plot colored by class label; tune UMAP hyperparameters (n_neighbors, min_dist) by grid search since min_dist significantly impacts separation of cell populations (e.g., on the Zheng sorted dataset). Use the --interactive mode to list layer names and indices along model depth and select which layer to visualize.

## Related tools

- **layerUMAP** (Command-line tool that exposes hidden-layer outputs along model depth, applies pooling/flatten reduction to rank>2 tensors, and renders UMAP layer-wise visualizations with an interactive layer-selection mode) — https://github.com/jingry/autoBioSeqpy
- **autoBioSeqpy** (Provides the data-processing modules to encode biological sequences (dict encoding, spcLen 100) and saves the Keras model JSON/weights/parameters artifacts consumed by layerUMAP) — https://github.com/jingry/autoBioSeqpy
- **UMAP** (Library used by layerUMAP to embed the reduced 2-D hidden-layer feature matrices into two-dimensional space for visualization)

## Examples

```
python running.py --dataType protein --dataEncodingType dict --dataTrainFilePaths examples/typeIII_secreted_effectors_prediction/data/train_pos.txt examples/typeIII_secreted_effectors_prediction/data/train_neg.txt --spcLen 100 --modelSaveName tmpMod. --weightSaveName tmpWeight.bin --paraSaveName parameters.txt
```

## Evaluation signals

- The reduced representation is a valid 2-D sample-by-feature matrix for every main-path layer examined.
- The module runs end-to-end on the shipped example CNN model (typeIII_secreted_effectors_prediction, protein_CNN_model.py).
- Each main-path layer yields a valid 2-D UMAP embedding and scatter plot colored by class label.
- The interactive mode correctly lists layer names and indices along model depth and the user selection matches the rendered layer.
- No API exists for capturing branch-layer outputs — verifying only main-path layers are processed, matching documented supported behavior.

## Limitations

- layerUMAP cannot capture the hidden outputs from layers in a branch of the built model; users must capture these manually using a Jupyter Notebook.
- Reported runtimes (approximately 30 s, excluding interaction time) are approximate and hardware-dependent without full specification.
- UMAP parameter min_dist significantly impacts the separation of cell populations, so the visualization outcome is sensitive to hyperparameter choices.

## Evidence
