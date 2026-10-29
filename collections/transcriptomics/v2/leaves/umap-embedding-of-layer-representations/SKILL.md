---
name: umap-embedding-of-layer-representations
description: Use when a trained deep learning model (e.g. a protein-sequence CNN built
  with autoBioSeqpy) needs its hidden-layer outputs visualized in 2-D UMAP space for
  layer-by-layer inspection, diagnosis or refinement of representations.
when_to_use_negative:
- The target layer lies in a branch of the built model — layerUMAP cannot capture
  branch-layer outputs; those must be captured manually via the Jupyter Notebook workaround
- The goal is only to predict labels for new sequences rather than inspect learned
  representations — use autoBioSeqpy's predicting.py instead
- The input is not a trained Keras model with autoBioSeqpy-encoded sequence data (e.g.
  no model JSON/weights/parameters.txt artifacts exist to load)
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: layerUMAP
  role: Command-line tool that exposes hidden-layer outputs along model depth and
    embeds the selected layer's features in 2-D UMAP space interactively
  repo: github.com/jingry/autoBioSeqpy
- name: autoBioSeqpy
  role: Encodes biological sequences into matrices (e.g. dict encoding at spcLen 100)
    and produces the model JSON, weights and parameters.txt artifacts that layerUMAP
    loads
  repo: github.com/jingry/autoBioSeqpy
- name: UMAP
  role: Library used by layerUMAP to embed reduced hidden-layer feature matrices into
    two dimensions for visualization
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2022.105530
    title: 'layerUMAP: A tool for visualizing and understanding deep learning models
      in biological sequence classification using UMAP'
  source_content_hash: sha256:0146111202aa83f64d321d8ba89a329765523fa06253a87d41bdf713782b25c4
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# umap-embedding-of-layer-representations

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

layerUMAP is a command-line tool, included in autoBioSeqpy, that embeds hidden-layer activations of trained Keras models into a 2-D UMAP space so users can inspect learned representations layer by layer and dissect model performance.

## When to use

Use when a trained deep learning model (e.g. a protein-sequence CNN built with autoBioSeqpy) needs its hidden-layer outputs visualized in 2-D UMAP space for layer-by-layer inspection, diagnosis or refinement of representations.

## When NOT to use

- The target layer lies in a branch of the built model — layerUMAP cannot capture branch-layer outputs; those must be captured manually via the Jupyter Notebook workaround
- The goal is only to predict labels for new sequences rather than inspect learned representations — use autoBioSeqpy's predicting.py instead
- The input is not a trained Keras model with autoBioSeqpy-encoded sequence data (e.g. no model JSON/weights/parameters.txt artifacts exist to load)

## Inputs

- Trained Keras model JSON file (e.g. tmpMod.json from --modelSaveName)
- Model weight file (e.g. tmpWeight.bin from --weightSaveName)
- Recorded training parameters file (e.g. tmpOut/parameters.txt from --paraSaveName)
- Labeled biological sequence data (e.g. protein FASTA train_pos.txt/train_neg.txt/test_pos.txt/test_neg.txt)
- User-selected layer (name or index along model depth) and reduction option ('max', 'min', 'avg' or 'flatten')
- UMAP hyperparameters (n_neighbors, min_dist)

## Outputs

- 2-D UMAP embedding of the selected hidden layer's reduced feature matrix
- Layer-wise scatter plot colored by class label, in the style of the paper's layer-by-layer dissection
- 2-D sample-by-feature matrix after pooling/flatten reduction of rank > 2 layer outputs

## How to apply

Load the trained Keras artifacts from an autoBioSeqpy run: the model JSON saved via --modelSaveName (tmpMod.json), the weights via --weightSaveName (tmpWeight.bin), and the recorded settings in tmpOut/parameters.txt. Encode input protein FASTA sequences with the autoBioSeqpy data-processing modules using the same encoding as training (dict encoding, --spcLen 100, as in the typeIII_secreted_effectors_prediction example). Traverse model.layers along the main path to build a multi-output model returning every hidden layer's activation, then reduce any output with rank > 2 to a 2-D sample-by-feature matrix using the user-selected reduction ('max', 'min', 'avg' pooling over the channel axis, or 'flatten'). Use the interactive option to list layer names and indices along model depth and select which layer's representation to embed, then run UMAP to 2-D and render a scatter plot colored by class label; parameters n_neighbors and min_dist can be grid-searched since e.g. min_dist significantly impacts separation of cell populations.

## Related tools

- **layerUMAP** (Command-line tool that exposes hidden-layer outputs along model depth and embeds the selected layer's features in 2-D UMAP space interactively) — github.com/jingry/autoBioSeqpy
- **autoBioSeqpy** (Encodes biological sequences into matrices (e.g. dict encoding at spcLen 100) and produces the model JSON, weights and parameters.txt artifacts that layerUMAP loads) — github.com/jingry/autoBioSeqpy
- **UMAP** (Library used by layerUMAP to embed reduced hidden-layer feature matrices into two dimensions for visualization)

## Examples

```
python running.py --dataType protein --dataEncodingType dict --dataTrainFilePaths examples/typeIII_secreted_effectors_prediction/data/train_pos.txt examples/typeIII_secreted_effectors_prediction/data/train_neg.txt --dataTrainLabel 1 0 --modelLoadFile examples/typeIII_secreted_effectors_prediction/model/protein_CNN_model.py --outSaveFolderPath tmpOut --spcLen 100 --modelSaveName tmpMod.json --weightSaveName tmpWeight.bin --paraSaveName parameters.txt
```

## Evaluation signals

- The module runs end-to-end on the shipped example CNN model (protein_CNN_model.py) from the typeIII_secreted_effectors_prediction example
- For each main-path layer, a valid 2-D UMAP embedding and scatter plot is produced; no API exposes branch-layer outputs, matching documented supported behavior
- Every rank > 2 layer output is reduced to a 2-D sample-by-feature matrix via the chosen pooling/flatten option before embedding
- The interactive mode lists layer names and indices along model depth consistent with the loaded model's architecture
- Plot points are colored by class label and UMAP hyperparameters (n_neighbors, min_dist) are recorded so parameter effects (e.g. min_dist on population separation) can be compared

## Limitations

- layerUMAP cannot capture hidden outputs from layers in a branch (if existing) of the built model; users must capture such outputs manually using the related Jupyter Notebook
- The actual text of command-line commands 1-5 is only referenced via Figure 2, not given inline, leaving a reproducibility gap
- Runtimes are approximate and hardware-dependent (e.g. ~30 s excluding interaction time) without full specification
- UMAP behavior is sensitive to parameters — e.g. the 'min_dist' parameter had a significant impact on the separation of cell populations

## Evidence
