---
name: gpu-deep-learning-training
description: Use when you must reproduce or adapt scBridge's iterative label transfer
  from annotated scRNA-seq to unlabeled scATAC-seq (h5ad AnnData) on a single GPU,
  e.g. the SNARE-seq mouse cortex benchmark.
when_to_use_negative:
- Input is paired (linked) multi-omics data, where paired-integration tools such as
  scAI or MOFA+ apply instead of scBridge's unpaired transfer learning
- Peak-by-cell scATAC-seq matrices without a gene activity transformation — peaks
  must first be converted to a gene activity matrix (e.g. via Signac)
- Scenarios requiring novel cell-type discovery mode (--novel_type) with its different
  heuristic stopping rule (>75% scATAC-seq cells annotated), which is outside the
  standard supervised label-transfer setup
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: scBridge
  role: The PyTorch model being trained for iterative multi-omics label transfer
  repo: github.com/XLearning-SCU/scBridge
- name: PyTorch 1.12.1
  role: Deep learning framework implementing scBridge's encoder f, classifier g, and
    prototype alignment losses
  repo: github.com/XLearning-SCU/scBridge
- name: Scikit-learn 1.1.1
  role: Provides the Gaussian Mixture Model used to score scATAC-seq cell reliability
    during iterative training
- name: CUDA 11.4
  role: GPU runtime for the single-GPU training runs (Nvidia RTX 3090)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:9197673aa04013d820348a61f015b3cded0b0b74a5e019077b6eb4fcfbbcd976
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CUDA
  - CUDA 11.4
  - PyTorch
  - PyTorch 1.12.1
  - Python
  - Scikit-learn 1.1.1
  - scBridge
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# gpu-deep-learning-training

## Summary

Train scBridge, a PyTorch-based heterogeneous transfer learning model, on a single CUDA-enabled GPU to iteratively integrate annotated scRNA-seq with unlabeled scATAC-seq data using a Gaussian Mixture reliability model. This skill covers the concrete training loop, hyperparameters, and hardware setup used to reproduce the SNARE-seq mouse cortex label-transfer result (71.95% accuracy).

## When to use

Use when you must reproduce or adapt scBridge's iterative label transfer from annotated scRNA-seq to unlabeled scATAC-seq (h5ad AnnData) on a single GPU, e.g. the SNARE-seq mouse cortex benchmark.

## When NOT to use

- Input is paired (linked) multi-omics data, where paired-integration tools such as scAI or MOFA+ apply instead of scBridge's unpaired transfer learning
- Peak-by-cell scATAC-seq matrices without a gene activity transformation — peaks must first be converted to a gene activity matrix (e.g. via Signac)
- Scenarios requiring novel cell-type discovery mode (--novel_type) with its different heuristic stopping rule (>75% scATAC-seq cells annotated), which is outside the standard supervised label-transfer setup

## Inputs

- Source scRNA-seq h5ad AnnData with cell-type annotations in obs['CellType']
- Target scATAC-seq h5ad AnnData (gene activity matrix, e.g. from GSE126074 SNARE-seq cortex)
- Common gene set shared between the scRNA-seq count matrix and scATAC-seq gene activity matrix

## Outputs

- Integrated target AnnData with transferred labels in .obs['Prediction']
- Per-cell reliability scores in .obs['Reliability']
- Label transfer accuracy and weighted F1 (expected ~71.95% accuracy, 42.26% weighted F1 on SNARE-seq cortex)

## How to apply

Clone github.com/XLearning-SCU/scBridge and create the conda environment from environment.yml; scBridge is implemented in Python using the PyTorch v.1.12.1 framework and requires exactly one GPU (paper ran on an Nvidia RTX 3090 with CUDA 11.4, Ubuntu 20.04). Prepare source (scRNA-seq) and target (scATAC-seq gene activity) h5ad AnnData files with source cell types in obs['CellType'], subset to common genes, and preprocess with the defaults Standard for source and TFIDF for target. Train via main.py with default hyperparameters: reliability_threshold=0.95, prototype_momentum=0.9, early_stop_acc=0.99, max_iteration=20, batch_size=512, train_epoch=20, learning_rate=5e-4, random_seed=2023; warm up the encoder f and classifier g with weighted cross-entropy loss, then model scATAC-seq cell reliability with a Gaussian Mixture (scikit-learn 1.1.1) on discriminability and confidence, selecting reliable cells (threshold 0.95) and repeating via cross-omics prototype alignment. Stop optimization early when annotated-data classification accuracy reaches 99%, then read .obs['Prediction'] and .obs['Reliability'] from the integrated target AnnData.

## Related tools

- **scBridge** (The PyTorch model being trained for iterative multi-omics label transfer) — github.com/XLearning-SCU/scBridge
- **PyTorch 1.12.1** (Deep learning framework implementing scBridge's encoder f, classifier g, and prototype alignment losses) — github.com/XLearning-SCU/scBridge
- **Scikit-learn 1.1.1** (Provides the Gaussian Mixture Model used to score scATAC-seq cell reliability during iterative training)
- **CUDA 11.4** (GPU runtime for the single-GPU training runs (Nvidia RTX 3090))

## Examples

```
git clone https://github.com/XLearning-SCU/scBridge && cd scBridge && conda env create -f environment.yml && conda activate scBridge && python main.py
```

## Evaluation signals

- First-iteration label transfer accuracy on SNARE-seq cortex should be ~60.11%, rising steadily across iterations to a final 71.95% (42.26% weighted F1)
- Training stops via early_stop_acc: annotated-data classification accuracy reaches 99%, or max_iteration=20 is hit
- Integrated target AnnData contains non-empty .obs['Prediction'] and .obs['Reliability'] columns with reliability values in [0,1]; reliable cells are those above reliability_threshold=0.95
- Accuracy computed against provided target annotations is stable across the fixed random_seed=2023 on a single GPU

## Limitations

- Requires a CUDA-enabled GPU; scBridge runs on a single GPU only
- Comparison results versus scJoint on protein data and the overall algorithm are deferred to Supplementary Notes 1 and 4, not in the main text
- Novel-type discovery mode uses a heuristic stopping rule (stop when more than 75% of scATAC-seq cells are selected as annotated), and guidance is only in Supplementary Note 2

## Evidence

- [discussion] PyTorch framework version: "scBridge is implemented in Python using the PyTorch41 framework, v.1.12.1."
- [discussion] GPU hardware: "All experiments are conducted on an Nvidia RTX 3090 GPU with CUDA 11.4, on the Ubuntu 20.04 OS."
- [discussion] Early stopping criterion: "we early stopped the network optimization when the cell classification accuracy of the annotated data reached 99%"
- [discussion] Reliability modeling: "we model the reliability of scATAC-seq cells with Gaussian Mixture based on their discriminability and confidence."
- [readme] Single GPU requirement: "Note: scBridge runs on a single GPU."
- [readme] Installation: "git clone https://github.com/XLearning-SCU/scBridge"
