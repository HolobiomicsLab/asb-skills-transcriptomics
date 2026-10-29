---
name: gcn-interaction-model-training
description: Use when you need to train DeepCCI's interaction model on processed ligand-receptor
  pair features and verify its generalizability for identifying significant CCIs via
  cross-validation metrics.
when_to_use_negative:
- Your goal is cell type identification/clustering rather than CCI inference — use
  DeepCCI's Cluster model (preprocess.py / Cluster.py) instead of the Interaction
  model
- Your data has no cell-type labels and no consensus significant-interaction labels
  from multiple CCI-prediction methods — majority-vote ground truth cannot be built
  and supervised training is not possible
- You only need downstream visualization of an already-computed CCI network — run
  Plot/Plot.py directly instead of retraining the interaction model
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: DeepCCI
  role: Framework providing Feature.py and Interaction_inference.py for pair-feature
    construction and interaction model inference/evaluation
  repo: github.com/JiangBioLab/DeepCCI
- name: conda
  role: Recommended environment manager to create the deepcciEnv virtual environment
    for the interaction model
- name: Python 3.7.4
  role: Runtime for training and inference scripts (Feature.py, Interaction_inference.py)
    on CUDA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad596
    title: 'DeepCCI: a deep learning framework for identifying cell–cell interactions
      from single-cell RNA sequencing data'
  source_content_hash: sha256:b8b5bd2e2836b76cd762d0f28895e1d8549fc8b48dfd55c879d1c7f110df577b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepCCI
  - Python
  - Python 3.7.4
  - conda
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# gcn-interaction-model-training

## Summary

Train and evaluate DeepCCI's ResNet+GCN ligand-receptor interaction model on scRNA-seq pair features, using repeated 5-fold cross-validation (5 folds x 20 repetitions) to quantify generalizability with F1, Recall, ACC, Precision, and AUC.

## When to use

Use when you need to train DeepCCI's interaction model on processed ligand-receptor pair features and verify its generalizability for identifying significant CCIs via cross-validation metrics.

## When NOT to use

- Your goal is cell type identification/clustering rather than CCI inference — use DeepCCI's Cluster model (preprocess.py / Cluster.py) instead of the Interaction model
- Your data has no cell-type labels and no consensus significant-interaction labels from multiple CCI-prediction methods — majority-vote ground truth cannot be built and supervised training is not possible
- You only need downstream visualization of an already-computed CCI network — run Plot/Plot.py directly instead of retraining the interaction model

## Inputs

- Processed ligand-receptor pair feature file from DeepCCI Feature.py (--label_mode True, SVD-reduced pair features)
- Consensus significant-interaction labels per ligand-receptor/cell-type pair (majority vote across constituent CCI-prediction methods)
- Ligand-receptor pair graph built with similarity threshold tau=0.95
- CUDA-capable GPU (--device cuda)

## Outputs

- Trained ResNet+GCN interaction model
- Predicted interaction outfile (generated in ~1-2 minutes per the README)
- Five cross-validation metrics (F1, Recall, ACC, Precision, AUC) aggregated over 5 folds x 20 repetitions, comparable to Fig. 3h-i benchmarks

## How to apply

Run DeepCCI's Interaction_model pipeline in a conda virtual environment (deepcciEnv, Python 3.7.4, pip install -r requirements.txt). Preprocess the scRNA-seq input with Feature.py using --label_mode True for labeled data and the correct --species (e.g., Human) to build the ligand-receptor pair feature file with SVD dimensionality reduction; construct the pair graph with similarity threshold tau=0.95. Aggregate ground-truth significant-interaction labels across constituent CCI-prediction methods by majority vote to obtain consensus labels per ligand-receptor/cell-type pair. Train the ResNet+GCN interaction model with Focal loss on a CUDA device to counter the significant/non-significant class imbalance, then run Interaction_inference.py --device cuda, repeating 5-fold cross-validation 20 times to yield 100 metric evaluations.

## Related tools

- **DeepCCI** (Framework providing Feature.py and Interaction_inference.py for pair-feature construction and interaction model inference/evaluation) — github.com/JiangBioLab/DeepCCI
- **conda** (Recommended environment manager to create the deepcciEnv virtual environment for the interaction model)
- **Python 3.7.4** (Runtime for training and inference scripts (Feature.py, Interaction_inference.py) on CUDA)

## Examples

```
cd Interaction_model; python Feature.py --label_mode True --species Human; python Interaction_inference.py --device cuda
```

## Evaluation signals

- The predicted interaction outfile is generated within the README's reported ~1-2 minute inference time
- Five-fold cross-validation repeated 20 times produces 100 metric evaluations that can be aggregated into mean F1, Recall, ACC, Precision, and AUC
- Reproduced mean metrics match the published benchmarks in Fig. 3h-i
- Pair graph edges reflect the tau=0.95 similarity threshold applied to ligand-receptor pair features
- Focal loss is active during training (class imbalance between significant and non-significant pairs is handled, not naive cross-entropy)

## Limitations

- Training and inference require a CUDA device; the interaction model commands assume GPU availability (--device cuda)
- The supervised evaluation depends on majority-vote consensus labels from constituent CCI-prediction methods, so label quality is bounded by the agreement of those methods
- Significant/non-significant interaction classes are imbalanced, requiring Focal loss; without it, metrics may be skewed toward the majority class

## Evidence

- [methods] Preprocess input files (Interaction model): "python Feature.py --label_mode True --species Human"
- [methods] Interaction Inference: "python Interaction_inference.py --device cuda"
- [intro] GCN-based framework definition: "DeepCCI is a graph convolutional network (GCN)-based deep learning framework for Cell-Cell Interactions inference from scRNA-seq data."
- [readme] Inference output and runtime: "The predicted interaction outfile will be provided. The predicted process will take about 1-2 minutes."
- [readme] Environment setup: "conda create -n deepcciEnv python=3.7.4 pip"
