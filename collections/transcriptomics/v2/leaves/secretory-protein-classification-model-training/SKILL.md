---
name: secretory-protein-classification-model-training
description: Use when the goal is to train protein-sequence deep learning classifiers
  on FASTA-derived encoded matrices and compare architecture performance (e.g. six-class
  bacterial secreted protein classification) with reported hyperparameters.
when_to_use_negative:
- Input is already a feature/encoding matrix rather than raw biological sequences
  — autoBioSeqpy's data encoding step would be redundant or incorrect
- The task is visualizing or diagnosing an already-trained model's hidden-layer representations
  rather than training classifiers — use layerUMAP instead
- Cell-type classification from single-cell gene expression data (e.g. Zheng sorted,
  Pancreas, CellBench) rather than protein sequence classification — different data
  modality and workflow
edam_operation: http://edamontology.org/operation_0474
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: autoBioSeqpy
  role: Standalone CLI (running.py, predicting.py) used to encode protein sequences
    into matrices (--dataEncodingType dict, --spcLen 100), train the five architectures,
    and generate held-out test predictions
  repo: github.com/jingry/autoBioSeqpy
- name: Keras
  role: Deep learning backend required by autoBioSeqpy; builds and trains the CNN,
    BiLSTM, BiGRU, CNN-BiLSTM and CNN-BiGRU models and stores model JSON and weights
- name: Anaconda3
  role: Dependency resolver for the required Python modules (re, numpy, importlib,
    matplotlib, sklearn, keras)
- name: layerUMAP
  role: Optional post-training command-line tool (integrated in autoBioSeqpy) using
    UMAP to inspect hidden-layer representations of the trained classifiers
  repo: github.com/jingry/autoBioSeqpy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2022.105530
    title: 'layerUMAP: A tool for visualizing and understanding deep learning models
      in biological sequence classification using UMAP'
  source_content_hash: sha256:92a896189ec54964d5aa2a1833366fa4ff0c123142f15285e2d88ea2fffbae3e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jingry/autoBioSeqpy
  tools:
  - Anaconda3
  - Keras
  - autoBioSeqpy
  - layerUMAP
  edam_operation: http://edamontology.org/operation_0474
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# secretory-protein-classification-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains and evaluates five deep learning architectures (CNN, BiLSTM, BiGRU, CNN-BiLSTM, CNN-BiGRU) in autoBioSeqpy for classifying Gram-negative bacterial secreted proteins (e.g. T1SP/T7SP six-class secretory-type classification), reproducing the reported accuracy comparison (CNN and CNN-BiGRU 97.9%, CNN-BiLSTM 97.7%, BiLSTM 97.3%, BiGRU 95.8%).

## When to use

Use when the goal is to train protein-sequence deep learning classifiers on FASTA-derived encoded matrices and compare architecture performance (e.g. six-class bacterial secreted protein classification) with reported hyperparameters.

## When NOT to use

- Input is already a feature/encoding matrix rather than raw biological sequences — autoBioSeqpy's data encoding step would be redundant or incorrect
- The task is visualizing or diagnosing an already-trained model's hidden-layer representations rather than training classifiers — use layerUMAP instead
- Cell-type classification from single-cell gene expression data (e.g. Zheng sorted, Pancreas, CellBench) rather than protein sequence classification — different data modality and workflow

## Inputs

- T1SP/T7SP protein FASTA sequences (train_pos.txt, train_neg.txt, test_pos.txt, test_neg.txt with labels 1/0)
- Keras model definition files (e.g. protein_CNN_model.py, protein_biLSTM_model.py) passed via --modelLoadFile
- Parameter file (parameters.txt) for reproducible invocation via --paraFile

## Outputs

- Trained keras model JSON (tmpMod.) and weights (tmpWeight.bin) per architecture
- Saved training parameters (tmpOut/parameters.txt)
- Per-model test predictions (indPredictions.txt) and accuracy figures via --savePrediction/--saveFig
- Aggregated five-architecture test accuracy comparison table (test_accuracy_comparison.csv)

## How to apply

Resolve Python dependencies (numpy, matplotlib, sklearn, keras) via Anaconda3 or pip, then encode the T1SP/T7SP protein FASTA sequences with running.py using --dataType protein, --dataEncodingType dict, and a fixed sequence length --spcLen 100. Train each architecture via running.py --modelLoadFile with the best hyperparameters: CNN with filter length 5 and 300 filters using global max pooling (global max pooling outperformed max pooling), and BiLSTM/BiGRU with input size 64 (hidden size 128 for the bidirectional layer); use --batch_size 60 --epochs 20 --shuffleDataTrain 1 --noGPU 0, saving the model as tmpMod. and weights as tmpWeight.bin. Repeat the training procedure 10 times per configuration and evaluate each hyperparameter/architecture using all 10 trained models, taking accuracy over the 10 models as the final score. Generate test predictions with predicting.py --paraFile tmpOut/parameters.txt on the held-out test files, then aggregate the five architectures' test accuracies into a comparison table (test_accuracy_comparison.csv).

## Related tools

- **autoBioSeqpy** (Standalone CLI (running.py, predicting.py) used to encode protein sequences into matrices (--dataEncodingType dict, --spcLen 100), train the five architectures, and generate held-out test predictions) — github.com/jingry/autoBioSeqpy
- **Keras** (Deep learning backend required by autoBioSeqpy; builds and trains the CNN, BiLSTM, BiGRU, CNN-BiLSTM and CNN-BiGRU models and stores model JSON and weights)
- **Anaconda3** (Dependency resolver for the required Python modules (re, numpy, importlib, matplotlib, sklearn, keras))
- **layerUMAP** (Optional post-training command-line tool (integrated in autoBioSeqpy) using UMAP to inspect hidden-layer representations of the trained classifiers) — github.com/jingry/autoBioSeqpy

## Examples

```
python running.py --dataType protein --dataEncodingType dict --dataTrainFilePaths train_pos.txt train_neg.txt --dataTrainLabel 1 0 --dataTestFilePaths test_pos.txt test_neg.txt --dataTestLabel 1 0 --modelLoadFile model/protein_CNN_model.py --outSaveFolderPath tmpOut --batch_size 60 --epochs 20 --spcLen 100 --shuffleDataTrain 1 --noGPU 0 --paraSaveName parameters.txt
```

## Evaluation signals

- test_accuracy_comparison.csv contains numeric test accuracies for all five architectures
- Unidirectional LSTM/GRU perform poorly compared to BiLSTM/BiGRU, consistent with the paper's reported findings
- CNN and CNN-BiGRU reach the highest test accuracy (~97.9%), followed by CNN-BiLSTM (97.7%), BiLSTM (97.3%) and BiGRU (95.8%) on the six-class task
- Final score per configuration is accuracy taken over 10 repeated training runs, not a single run
- predicting.py runs successfully on tmpOut/parameters.txt without re-specifying encoding parameters

## Limitations

- layerUMAP cannot capture hidden outputs from layers in a branch (if existing) of the built model; branch outputs must be captured manually via Jupyter Notebook
- Runtimes are approximate and hardware-dependent without full specification
- The exact text of command-line commands 1–5 is only referenced via Figure 2, not given inline (reproducibility gap)
- Training is stochastic — each hyperparameter must be evaluated over 10 trained models rather than a single fit

## Evidence

- [results] CNN and CNN-BiGRU achieved the highest test accuracy of 97.9%, followed by CNN-BiLSTM at 97.7%, BiLSTM at 97.3%, and BiGRU at 95.8%: [evidence span withheld — anchor status: not_found]
