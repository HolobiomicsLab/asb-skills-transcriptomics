---
name: hidden-layer-activation-extraction
description: Use when a trained Keras model (tmpMod. + tmpWeight.bin + parameters.txt)
  is available and you need to inspect or validate what its hidden layers learned,
  e.g. UMAP plots of last-hidden-layer activations colored by class labels.
when_to_use_negative:
- The target layer lies in a branch of the built model — layerUMAP cannot capture
  branch-layer outputs; use the Jupyter Notebook workaround instead
- You only need sequence encoding into matrices without model inference — autoBioSeqpy
  can be used directly as an encoding module without UMAP projection
- You need fully scripted command text inline — commands 1-5 are only reproduced in
  Figure 2, so a purely non-interactive reproduction must recover them from the figure
edam_operation: http://edamontology.org/operation_0337
tools:
- name: layerUMAP
  role: Command-line tool included in autoBioSeqpy that captures hidden-layer activations
    and projects them with UMAP; supports grid search over n_neighbors, min_dist and
    distance metrics via CLI flags
  repo: https://github.com/jingry/autoBioSeqpy
- name: autoBioSeqpy
  role: Loads the trained Keras model, weights (tmpWeight.bin) and parameters.txt,
    and encodes protein/DNA/RNA FASTA inputs into matrices with dict encoding and
    spcLen 100
  repo: https://github.com/jingry/autoBioSeqpy
- name: UMAP
  role: Library performing the 2D non-linear projection of the extracted hidden-layer
    representations
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2022.105530
    title: 'layerUMAP: A tool for visualizing and understanding deep learning models
      in biological sequence classification using UMAP'
  source_content_hash: sha256:f67c0984fb592e01e53a5f539bac2b597bde7d95bfd528b34731aecabf30a565
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# hidden-layer-activation-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract and visualize the activations of hidden layers of trained deep learning models (Keras CNN, RNN, RFCN) using layerUMAP, a command-line tool that integrates autoBioSeqpy with the UMAP library, projecting learned high-level representations of biological sequences or single-cell expression data into 2D.

## When to use

Use when a trained Keras model (tmpMod. + tmpWeight.bin + parameters.txt) is available and you need to inspect or validate what its hidden layers learned, e.g. UMAP plots of last-hidden-layer activations colored by class labels.

## When NOT to use

- The target layer lies in a branch of the built model — layerUMAP cannot capture branch-layer outputs; use the Jupyter Notebook workaround instead
- You only need sequence encoding into matrices without model inference — autoBioSeqpy can be used directly as an encoding module without UMAP projection
- You need fully scripted command text inline — commands 1-5 are only reproduced in Figure 2, so a purely non-interactive reproduction must recover them from the figure

## Inputs

- Trained Keras model JSON (tmpMod.) with weights (tmpWeight.bin) and recorded training parameters (tmpOut/parameters.txt)
- Test sequences as FASTA text files (e.g. examples/typeIII_secreted_effectors_prediction/data/test_pos.txt and test_neg.txt)
- Single-cell gene expression data (e.g. Zheng sorted dataset) with cell ontology labels from the original publication

## Outputs

- 2D UMAP embedding scatter plot of last-hidden-layer activations colored by class label (e.g. umap_last_hidden_layer.png)
- Layer-by-layer interactive visualization of hidden-layer outputs along model depth
- Composite UMAP metric-comparison figure (umap_metric_comparison.png) when run across ten distance metrics

## How to apply

Load the trained model saved as tmpMod. together with its weights tmpWeight.bin and the training-record file parameters.txt via autoBioSeqpy. Feed test data — protein FASTA sequences (test_pos.txt/test_neg.txt, --dataType protein --dataEncodingType dict --spcLen 100) or single-cell gene expression matrices — through the network and capture the last hidden layer activations with the layerUMAP command-line tool (command-line command 2). Project the activations into 2D with the UMAP library, grid-searching n_neighbors and min_dist via command-line command 4 since, for example, min_dist significantly impacted separation of cell populations on the Zheng sorted dataset. Render the embedding as a scatter plot colored by true labels (e.g. T1SS-T7SS secretory types or cell ontology labels), optionally repeating the projection per distance metric (chebyshev, braycurtis, canberra, correlation, cosine, euclidean, mahalanobis, manhattan, minkowski, wminkowski) and saving each figure with saveFig enabled. To trace representations layer by layer through the depth of the model, use the interactive option (command-line command 3).

## Related tools

- **layerUMAP** (Command-line tool included in autoBioSeqpy that captures hidden-layer activations and projects them with UMAP; supports grid search over n_neighbors, min_dist and distance metrics via CLI flags) — https://github.com/jingry/autoBioSeqpy
- **autoBioSeqpy** (Loads the trained Keras model, weights (tmpWeight.bin) and parameters.txt, and encodes protein/DNA/RNA FASTA inputs into matrices with dict encoding and spcLen 100) — https://github.com/jingry/autoBioSeqpy
- **UMAP** (Library performing the 2D non-linear projection of the extracted hidden-layer representations)

## Examples

```
python predicting.py --paraFile tmpOut/parameters.txt --dataTestFilePaths examples/typeIII_secreted_effectors_prediction/data/test_pos.txt --predictionSavePath tmpout/indPredictions.txt
```

## Evaluation signals

- umap_last_hidden_layer.png exists in the designated output folder and points are colored by the true labels (T1SS-T7SS for the secretory-type task)
- Expert visual review confirms the six secretory types form distinguishable clusters in the 2D projection, consistent with the reported CNN classification accuracy
- In the ten-metric comparison on the Zheng sorted dataset, nine metrics (all but wminkowski) yield separated cell-population clusters while wminkowski alone yields mixed, non-separated populations
- Best CNN configuration matches the paper's grid search: filter length = 5 with 300 filters, using global max pooling rather than max pooling
- The command completes in roughly 30 s (excluding interaction time) on typical hardware, so large deviations indicate environment or path problems

## Limitations

- layerUMAP cannot capture hidden outputs from layers located in a branch of the built model; branch outputs must be extracted manually via the related Jupyter Notebook
- The runtime of the visualization command is approximate and hardware-dependent without full hardware specification in the paper
- The literal text of command-line commands 1-5 is only shown in Figure 2, creating a reproducibility gap for fully scripted reproduction

## Evidence
