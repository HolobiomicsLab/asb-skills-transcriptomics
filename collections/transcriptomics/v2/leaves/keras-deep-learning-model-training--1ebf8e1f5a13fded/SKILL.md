---
name: keras-deep-learning-model-training
description: Use when you need to train a Keras deep learning classifier (CNN, BiLSTM/BiGRU,
  RFCN) on fixed-length biological sequence data or scRNA-seq gene expression samples
  and obtain saved model, weights, parameters, predictions and figures.
when_to_use_negative:
- You only want feature encoding of FASTA sequences into matrices without modeling
  — autoBioSeqpy can be used as a module in a Python script or Jupyter notebook instead
  of running.py
- You need to visualize hidden-layer outputs of an already-trained model — use layerUMAP,
  not the training workflow
- Input samples are not fixed-length or cannot be encoded by the dict encoding at
  the chosen --spcLen
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: autoBioSeqpy
  role: Python framework that converts sequences into matrices and trains/predicts
    Keras models via running.py and predicting.py; layerUMAP integrates it for visualization
  repo: github.com/jingry/autoBioSeqpy
- name: Keras
  role: Deep learning backend required as a dependency; trained models are saved as
    Keras model JSON plus weight files
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2022.105530
    title: 'layerUMAP: A tool for visualizing and understanding deep learning models
      in biological sequence classification using UMAP'
  source_content_hash: sha256:0e06e3030ee500e39c2e5e93c17f6f288db4853e0f113bd2e320ea2e22e54d43
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# keras-deep-learning-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train Keras-based deep learning models (CNN, BiLSTM, BiGRU, RFCN and related architectures) for biological sequence and single-cell gene-expression classification using autoBioSeqpy's running.py CLI, with dict encoding of fixed-length samples into numeric matrices.

## When to use

Use when you need to train a Keras deep learning classifier (CNN, BiLSTM/BiGRU, RFCN) on fixed-length biological sequence data or scRNA-seq gene expression samples and obtain saved model, weights, parameters, predictions and figures.

## When NOT to use

- You only want feature encoding of FASTA sequences into matrices without modeling — autoBioSeqpy can be used as a module in a Python script or Jupyter notebook instead of running.py
- You need to visualize hidden-layer outputs of an already-trained model — use layerUMAP, not the training workflow
- Input samples are not fixed-length or cannot be encoded by the dict encoding at the chosen --spcLen

## Inputs

- Training data files (e.g. train_pos.txt/train_neg.txt) with corresponding class labels (--dataTrainLabel, e.g. 1 0)
- Test data files (test_pos.txt/test_neg.txt) with labels
- Model definition Python file passed via --modelLoadFile (e.g. protein_CNN_model.py, protein_biLSTM_model.py, RFCN model)
- Sample length parameter --spcLen (e.g. 100) and encoding type (--dataEncodingType dict)
- A parameters text file usable with --paraFile as an alternative to inline arguments

## Outputs

- Keras model file (--modelSaveName tmpMod.) and weights (--weightSaveName tmpWeight.bin)
- parameters.txt recording training parameters, reusable with predicting.py
- Per-sample predictions (--savePrediction 1) and figures (--saveFig 1) in the --outSaveFolderPath (e.g. tmpOut)
- Overall accuracy metrics on held-out test sets (e.g. 98.09% / 99.78% / 100% for Zheng sorted / Pancreas / CellBench)

## How to apply

Resolve Python dependencies (numpy, matplotlib, sklearn, keras) via Anaconda3 or pip, then set the autoBioSeqpy folder on the search path or run from its root directory. Prepare paired training and test files with class labels, and encode fixed-length samples (controlled by --spcLen, e.g. 100) into numeric matrices with --dataEncodingType dict. Train the architecture loaded via --modelLoadFile using running.py with batch_size 60, epochs 20, training data shuffled (--shuffleDataTrain 1) and GPU enabled (--noGPU 0); for the RFCN scRNA-seq case study this yielded overall accuracies of 98.09% (Zheng sorted), 99.78% (Pancreas) and 100% (CellBench). Save the Keras model JSON, weights (tmpWeight.bin) and parameters (parameters.txt) to --outSaveFolderPath, then evaluate held-out test sets with predicting.py --paraFile tmpOut/parameters.txt. For prediction-only use, the necessary files are the saved model, weights, and parameters recorded in parameters.txt.

## Related tools

- **autoBioSeqpy** (Python framework that converts sequences into matrices and trains/predicts Keras models via running.py and predicting.py; layerUMAP integrates it for visualization) — github.com/jingry/autoBioSeqpy
- **Keras** (Deep learning backend required as a dependency; trained models are saved as Keras model JSON plus weight files)

## Examples

```
python running.py --dataEncodingType dict --dataTrainFilePaths train_pos.txt train_neg.txt --dataTrainLabel 1 0 --modelLoadFile model/protein_CNN_model.py --outSaveFolderPath tmpOut --batch_size 60 --epochs 20 --spcLen 100 --shuffleDataTrain 1 --noGPU 0
```

## Evaluation signals

- running.py --help shows the help document without error, confirming dependencies are resolved
- Output folder contains tmpMod., tmpWeight.bin, parameters.txt, predictions and figures when --savePrediction 1 and --saveFig 1 are set
- predicting.py --paraFile tmpOut/parameters.txt runs on held-out test files without additional encoding arguments
- Overall test accuracy is in the reported range (RFCN: 98.09% Zheng sorted, 99.78% Pancreas, 100% CellBench)
- Saved parameters.txt fully reconstructs encoding/model settings so predictions reproduce without re-specifying training parameters

## Limitations

- All dependent modules (re, numpy, importlib, matplotlib, sklearn, keras) must be installed; Anaconda3 version 2 is not suggested
- Too many parameters to write on the command line — users must supply a parameters.txt file via --paraFile for complex runs
- Pure HTML and PDF versions of the tutorial are read-only with no interaction; only the Jupyter notebook is interactive

## Evidence
