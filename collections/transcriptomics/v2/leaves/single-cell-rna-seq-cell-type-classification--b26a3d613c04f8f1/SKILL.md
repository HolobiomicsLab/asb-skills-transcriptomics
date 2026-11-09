---
name: single-cell-rna-seq-cell-type-classification
description: Use when labeled scRNA-seq expression datasets (e.g. Zheng sorted, Pancreas,
  CellBench) need supervised cell-type classification with a deep learning model,
  and when overall test accuracy must be benchmarked or reproduced.
when_to_use_negative:
- The model's architecture contains branches — layerUMAP cannot capture hidden outputs
  from layers in a branch of the built model, so layer-wise inspection is not applicable
- The goal is unsupervised clustering without class labels — the skill requires labeled
  training/test files with --dataTrainLabel/--dataTestLabel
- Sequences/samples are variable-length and cannot be truncated to the fixed --spcLen
  required by dict encoding
edam_operation: http://edamontology.org/operation_0478
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: autoBioSeqpy
  role: Encodes samples into numeric matrices via dict encoding and trains/evaluates
    the RFCN through running.py and predicting.py
  repo: github.com/jingry/autoBioSeqpy
- name: Keras
  role: Backend framework in which the RFCN model is defined, trained (batch_size
    60, epochs 20, GPU enabled) and saved
- name: layerUMAP
  role: Command-line tool included in autoBioSeqpy that uses UMAP to visualize last-hidden-layer
    cell representations for diagnosing RFCN performance layer by layer
  repo: github.com/jingry/autoBioSeqpy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2022.105530
    title: 'layerUMAP: A tool for visualizing and understanding deep learning models
      in biological sequence classification using UMAP'
  source_content_hash: sha256:837a639df158a83f2c235f7a9977825f832dba37fb83b850df058e5841a43332
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jingry/autoBioSeqpy
  tools:
  - Keras
  - autoBioSeqpy
  - layerUMAP
  edam_operation: http://edamontology.org/operation_0478
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# single-cell-rna-seq-cell-type-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify cell types from scRNA-seq gene-expression profiles using the RFCN residual fully connected network trained in autoBioSeqpy/Keras, reproducing reported overall accuracies of 98.09% (Zheng sorted), 99.78% (Pancreas) and 100% (CellBench). Fixed-length samples are dict-encoded into numeric matrices and modeled via command line with reproducible saved parameters.

## When to use

Use when labeled scRNA-seq expression datasets (e.g. Zheng sorted, Pancreas, CellBench) need supervised cell-type classification with a deep learning model, and when overall test accuracy must be benchmarked or reproduced.

## When NOT to use

- The model's architecture contains branches — layerUMAP cannot capture hidden outputs from layers in a branch of the built model, so layer-wise inspection is not applicable
- The goal is unsupervised clustering without class labels — the skill requires labeled training/test files with --dataTrainLabel/--dataTestLabel
- Sequences/samples are variable-length and cannot be truncated to the fixed --spcLen required by dict encoding

## Inputs

- Training and test scRNA-seq expression matrices as text files (e.g. train_pos.txt/train_neg.txt style) with per-file class labels (--dataTrainLabel/--dataTestLabel)
- RFCN architecture definition file passed via --modelLoadFile (Keras model .py)
- Trained model artifacts for prediction: Keras model JSON, weight .bin, and parameters.txt

## Outputs

- Overall cell-type classification accuracy per benchmark dataset (Zheng sorted, Pancreas, CellBench)
- Saved Keras model JSON, weights, and parameters.txt in the output folder
- Per-sample predictions (--savePrediction 1) and figures (--saveFig 1)
- Optional 2D UMAP visualizations of hidden-layer cell representations from layerUMAP

## How to apply

Install the Python dependencies required by autoBioSeqpy (re, numpy, importlib, matplotlib, sklearn, keras), most easily via anaconda3 or pip, and set the library path via sys.path.append if working outside the repo root. Load training and test expression files with their class labels using --dataTrainFilePaths/--dataTestFilePaths with --dataTrainLabel/--dataTestLabel, and encode fixed-length samples into numeric matrices with dict encoding (--dataEncodingType dict) at the length given by --spcLen. Train the RFCN by running 'python running.py' with the model definition loaded through --modelLoadFile, using batch_size 60, epochs 20, --shuffleDataTrain 1, and --noGPU 0 to enable GPU. Save the Keras model JSON, weights, and training parameters (--modelSaveName, --weightSaveName, --paraSaveName) into --outSaveFolderPath together with predictions and figures (--savePrediction 1, --saveFig 1), then evaluate on held-out test sets with 'python predicting.py --paraFile tmpOut/parameters.txt' and compute overall accuracy per dataset. Optionally inspect learned cell representations of the last hidden layer in 2D UMAP space using layerUMAP, tuning n_neighbors and min_dist by grid search to verify class separation.

## Related tools

- **autoBioSeqpy** (Encodes samples into numeric matrices via dict encoding and trains/evaluates the RFCN through running.py and predicting.py) — github.com/jingry/autoBioSeqpy
- **Keras** (Backend framework in which the RFCN model is defined, trained (batch_size 60, epochs 20, GPU enabled) and saved)
- **layerUMAP** (Command-line tool included in autoBioSeqpy that uses UMAP to visualize last-hidden-layer cell representations for diagnosing RFCN performance layer by layer) — github.com/jingry/autoBioSeqpy

## Examples

```
python running.py --dataEncodingType dict --dataTrainFilePaths train.txt --dataTrainLabel 1 0 --dataTestFilePaths test.txt --dataTestLabel 1 0 --modelLoadFile model/rfcn_model.py --outSaveFolderPath tmpOut --savePrediction 1 --saveFig 1 --batch_size 60 --epochs 20 --spcLen 100 --shuffleDataTrain 1 --modelSaveName tmpMod. --weightSaveName tmpWeight.bin --noGPU 0 --paraSaveName parameters.txt
```

## Evaluation signals

- Overall accuracies should match or approach the reported values: 98.09% on Zheng sorted, 99.78% on Pancreas, and 100% on CellBench
- Output folder must contain parameters.txt, the saved Keras model (--modelSaveName tmpMod.), weights (--weightSaveName tmpWeight.bin), per-sample predictions, and figures
- 'python running.py --help' must display the help document without error before training (installation check)
- Re-running predicting.py with tmpOut/parameters.txt on held-out test files should reproduce predictions without retraining
- UMAP plots of the last hidden layer should show visible separation of cell populations; min_dist and n_neighbors should be tuned by grid search if separation is poor

## Limitations

- layerUMAP cannot capture the hidden outputs from layers in a branch (if existing) of the built model; branch outputs must be captured manually using the related Jupyter Notebook
- Reported runtimes (e.g. approximately 30 s per command) are approximate and hardware-dependent and are not fully specified
- If autoBioSeqpy is not installed in the Python search path, the folder location must be added manually via sys.path.append before importing modules

## Evidence
