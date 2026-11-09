---
name: deep-learning-hidden-layer-representation-analysis
description: Use when you have a trained Keras model from autoBioSeqpy (model JSON,
  weight bin, parameters.txt) plus encoded test sequences, and need to inspect learned
  hidden-layer representations via UMAP embeddings.
when_to_use_negative:
- The target layers sit in a branch of the built model — layerUMAP cannot capture
  hidden outputs from branch layers; manual capture via Jupyter Notebook is the workaround
- The goal is training or hyperparameter grid search (e.g. filter length 3–13, number
  of filters) rather than visualizing an already-trained model
- No trained model artifacts (tmpMod., tmpWeight.bin, parameters.txt) exist — the
  raw sequences must first be modeled with autoBioSeqpy running.py
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: layerUMAP
  role: Command-line tool included in autoBioSeqpy that applies UMAP to hidden-layer
    outputs for layer-by-layer dissection of trained models
  repo: github.com/jingry/autoBioSeqpy
- name: autoBioSeqpy
  role: Host framework that encodes FASTA sequences into matrices, trains/predicts
    keras models, and provides the standalone scripts (running.py, predicting.py)
    whose saved artifacts layerUMAP consumes
  repo: github.com/jingry/autoBioSeqpy
- name: UMAP
  role: Dimensionality reduction library wrapped by layerUMAP to embed each hidden
    layer's representations in 2D
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2022.105530
    title: 'layerUMAP: A tool for visualizing and understanding deep learning models
      in biological sequence classification using UMAP'
  source_content_hash: sha256:19434df881c733aca5329854488625f4754df6a77a2e8801d609df5b344b12ef
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
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# deep-learning-hidden-layer-representation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Layer-by-layer dissection of trained autoBioSeqpy deep learning models using layerUMAP, which applies UMAP to each hidden layer's outputs (including max/min/avg reductions of convolution tensors) to visualize how sequence class clusters organize along model depth.

## When to use

Use when you have a trained Keras model from autoBioSeqpy (model JSON, weight bin, parameters.txt) plus encoded test sequences, and need to inspect learned hidden-layer representations via UMAP embeddings.

## When NOT to use

- The target layers sit in a branch of the built model — layerUMAP cannot capture hidden outputs from branch layers; manual capture via Jupyter Notebook is the workaround
- The goal is training or hyperparameter grid search (e.g. filter length 3–13, number of filters) rather than visualizing an already-trained model
- No trained model artifacts (tmpMod., tmpWeight.bin, parameters.txt) exist — the raw sequences must first be modeled with autoBioSeqpy running.py

## Inputs

- Trained keras model JSON (tmpMod.) and weights (tmpWeight.bin) saved by autoBioSeqpy running.py
- Recorded training parameters file (parameters.txt)
- Protein/biological test sequences encoded with dict encoding at spcLen 100
- autoBioSeqpy model load file (e.g. protein_CNN_model.py)

## Outputs

- One UMAP embedding figure (PNG) per dissected hidden layer
- Interactive inter-layer evolution plot showing cluster separation along model depth
- UMAP embeddings of convolution output tensor reduced over 'Filtered sequence length' and 'Channel' directions (max, min, avg)
- Saved figures in --outSaveFolderPath

## How to apply

Load the trained keras model architecture (tmpMod.), its weights (tmpWeight.bin), and the recorded training parameters (parameters.txt) through autoBioSeqpy, together with protein test sequences encoded with dict encoding at spcLen 100. Run layerUMAP — the command-line tool in autoBioSeqpy that wraps the UMAP library — to capture the outputs of each hidden layer of the CNN for the test sequences, then apply UMAP to each layer's representation and assemble the interactive inter-layer evolution plot showing how class clusters progressively separate along model depth. For the convolutional layer output tensor, collapse it separately over the 'Filtered sequence length' and 'Channel' directions using max, min, and avg reductions, and embed each reduced view with UMAP to visualize the learned convolutional features (the paper's best CNN used filter length = 5 and 300 filters). Write all figures with --saveFig 1 into the directory given by --outSaveFolderPath.

## Related tools

- **layerUMAP** (Command-line tool included in autoBioSeqpy that applies UMAP to hidden-layer outputs for layer-by-layer dissection of trained models) — github.com/jingry/autoBioSeqpy
- **autoBioSeqpy** (Host framework that encodes FASTA sequences into matrices, trains/predicts keras models, and provides the standalone scripts (running.py, predicting.py) whose saved artifacts layerUMAP consumes) — github.com/jingry/autoBioSeqpy
- **UMAP** (Dimensionality reduction library wrapped by layerUMAP to embed each hidden layer's representations in 2D)

## Examples

```
python running.py --dataType protein --dataEncodingType dict --dataTrainFilePaths examples/typeIII_secreted_effectors_prediction/data/train_pos.txt examples/typeIII_secreted_effectors_prediction/data/train_neg.txt --dataTrainLabel 1 0 --dataTestFilePaths examples/typeIII_secreted_effectors_prediction/data/test_pos.txt examples/typeIII_secreted_effectors_prediction/data/test_neg.txt --dataTestLabel 1 0 --modelLoadFile examples/typeIII_secreted_effectors_prediction/model/protein_CNN_model.py --verbose 1 --outSaveFolderPath tmpOut --savePrediction 1 --saveFig 1 --batch_size 60 --epochs 20 --spcLen 100 --shuffleDataTrain 1 --modelSaveName tmpMod. --weightSaveName tmpWeight.bin --noGPU 0 --paraSaveName parameters.txt
```

## Evaluation signals

- A UMAP figure is produced for every dissected hidden layer of the model
- Each saved PNG in --outSaveFolderPath exists and is non-empty (run with --saveFig 1)
- The inter-layer evolution plot reproduces progressively separated class clusters along model depth, as reported for the CNN in Figure 2 of the layerUMAP paper
- For the convolutional layer, three separate UMAP embeddings are generated for the max, min, and avg reductions over both the 'Filtered sequence length' and 'Channel' directions
- The loaded model artifacts (tmpMod., tmpWeight.bin, parameters.txt) are consistent, i.e. predicting.py can reload them with --paraFile tmpOut/parameters.txt without error

## Limitations

- layerUMAP cannot capture hidden outputs from layers in a branch (if existing) of the built model; users can only capture the outputs manually using the related Jupyter Notebook
- The actual text of the command-line commands for layerUMAP is only referenced via Figure 2 of the paper, not given inline (reproducibility gap)
- Runtimes reported are approximate and hardware-dependent; the grid search over UMAP parameters n_neighbors and min_dist (e.g. min_dist strongly affects cell population separation on the Zheng sorted dataset) adds interaction-time cost

## Evidence
