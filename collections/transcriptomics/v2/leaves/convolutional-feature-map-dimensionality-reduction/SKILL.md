---
name: convolutional-feature-map-dimensionality-reduction
description: 'Use when dissecting a trained CNN for biological sequence classification:
  you want to visualize what convolutional filters learned by reducing the conv-layer
  output tensor over ''Filtered sequence length'' and ''Channel'' axes and embedding
  with UMAP.'
when_to_use_negative:
- The model contains no convolutional layer (e.g. BiLSTM/BiGRU-only architectures)
  — there is no conv output tensor to collapse over 'Filtered sequence length'/'Channel'
- The layer of interest sits inside a branch of the built model — layerUMAP cannot
  capture hidden outputs from branch layers; use the Jupyter Notebook workaround instead
- You only need final predictions or training/prediction runs without representation
  inspection — plain running.py/predicting.py suffices
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: layerUMAP
  role: Command-line tool included in autoBioSeqpy that captures hidden-layer outputs
    of the trained model and applies UMAP to each layer, including reduced convolutional
    feature views
  repo: github.com/jingry/autoBioSeqpy
- name: autoBioSeqpy
  role: Encodes biological sequences into matrices (dict encoding, spcLen 100), loads
    the model architecture (tmpMod.), weights (tmpWeight.bin) and parameters (parameters.txt),
    and hosts layerUMAP
  repo: github.com/jingry/autoBioSeqpy
- name: UMAP
  role: Dimensionality-reduction library wrapped by layerUMAP to embed each hidden-layer
    (and reduced convolutional) representation into 2D
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2022.105530
    title: 'layerUMAP: A tool for visualizing and understanding deep learning models
      in biological sequence classification using UMAP'
  source_content_hash: sha256:85fe97e31464696413be7f8c081138aa0c37453fafc328eefebb7a841fdbeeca
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# convolutional-feature-map-dimensionality-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reduces the high-dimensional output tensor of a convolutional layer in a trained autoBioSeqpy CNN by collapsing it separately over the 'Filtered sequence length' and 'Channel' directions (max, min, avg reductions) and embedding each reduced view with UMAP via layerUMAP, exposing the sequence features the filters have learned.

## When to use

Use when dissecting a trained CNN for biological sequence classification: you want to visualize what convolutional filters learned by reducing the conv-layer output tensor over 'Filtered sequence length' and 'Channel' axes and embedding with UMAP.

## When NOT to use

- The model contains no convolutional layer (e.g. BiLSTM/BiGRU-only architectures) — there is no conv output tensor to collapse over 'Filtered sequence length'/'Channel'
- The layer of interest sits inside a branch of the built model — layerUMAP cannot capture hidden outputs from branch layers; use the Jupyter Notebook workaround instead
- You only need final predictions or training/prediction runs without representation inspection — plain running.py/predicting.py suffices

## Inputs

- Trained keras model architecture file (e.g. tmpMod.)
- Saved model weights (tmpWeight.bin)
- Recorded training parameters file (parameters.txt)
- Protein (or DNA/RNA) test sequences encoded with dict encoding at spcLen 100 via autoBioSeqpy data processing

## Outputs

- UMAP scatter plots (PNG) of each reduced convolutional-feature view (max, min, avg over 'Filtered sequence length' and 'Channel' directions)
- Interactive inter-layer evolution plot of hidden-layer representations along model depth
- Saved figures written to --outSaveFolderPath when --saveFig 1

## How to apply

Load the trained keras model architecture (tmpMod.), its weights (tmpWeight.bin) and the recorded training parameters (parameters.txt) through autoBioSeqpy, together with protein test sequences encoded with dict encoding at spcLen 100. Run layerUMAP, the command-line tool in autoBioSeqpy that wraps the UMAP library, to capture the outputs of each hidden layer of the CNN for the test sequences. For the convolutional layer output tensor specifically, collapse it separately over the 'Filtered sequence length' and 'Channel' directions using max, min, and avg reductions, then apply UMAP to each reduced view to visualize the learned convolutional features. Assemble the interactive inter-layer evolution plot showing how sequence class clusters organize along the depth of the model. Write all generated figures with --saveFig 1 into the directory given by --outSaveFolderPath. Note the paper's CNN used filter length = 5 with 300 filters and global max pooling, which shapes the tensor dimensions being reduced.

## Related tools

- **layerUMAP** (Command-line tool included in autoBioSeqpy that captures hidden-layer outputs of the trained model and applies UMAP to each layer, including reduced convolutional feature views) — github.com/jingry/autoBioSeqpy
- **autoBioSeqpy** (Encodes biological sequences into matrices (dict encoding, spcLen 100), loads the model architecture (tmpMod.), weights (tmpWeight.bin) and parameters (parameters.txt), and hosts layerUMAP) — github.com/jingry/autoBioSeqpy
- **UMAP** (Dimensionality-reduction library wrapped by layerUMAP to embed each hidden-layer (and reduced convolutional) representation into 2D)

## Examples

```
python running.py --dataType protein --dataEncodingType dict --spcLen 100 --modelSaveName tmpMod. --weightSaveName tmpWeight.bin --paraSaveName parameters.txt --saveFig 1 --outSaveFolderPath tmpOut
```

## Evaluation signals

- A UMAP figure (PNG) is produced for every dissected hidden layer, and every saved PNG in --outSaveFolderPath exists and is non-empty
- The inter-layer evolution plot reproduces progressively separated class clusters along the depth of the CNN, consistent with Figure 2 of the layerUMAP paper
- All three reductions (max, min, avg) of the convolution output tensor are embedded separately over both the 'Filtered sequence length' and 'Channel' collapsed directions
- The model/weights/parameters triple (tmpMod., tmpWeight.bin, parameters.txt) reloads without error through running.py/predicting.py, confirming the dissected model matches the trained one
- UMAP parameter choices (n_neighbors, min_dist) are documented if the grid search (command 4) was applied to tune the embedding

## Limitations

- layerUMAP cannot capture hidden outputs from layers in a branch (if existing) of the built model; branch-layer outputs must be captured manually via the related Jupyter Notebook
- The actual text of the command-line commands for layer-by-layer dissection is only referenced via Figure 2, not given inline in the paper (reproducibility gap)
- Reported runtimes (approximately 30 s, excluding interaction time) are approximate and hardware-dependent; UMAP results depend on parameters such as min_dist, which had a significant impact on cell-population separation on the Zheng sorted dataset

## Evidence

- [readme] model files and parameters saved for reloading: [evidence span withheld — anchor status: not_found]
