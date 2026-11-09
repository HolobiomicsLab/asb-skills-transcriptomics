---
name: neural-network-classifier-training
description: Use when a labeled reference scRNA-seq matrix and an unlabeled query
  matrix must be processed through scNovel's two-stage pipeline, i.e. a trained cell-type
  probability classifier is needed before novel cell detection with threshold lambda
  = 0.5.
when_to_use_negative:
- Input is already normalized/log-transformed feature embeddings rather than raw count
  matrices — reapplying normalize_total(target_sum=1e4) and log1p would double-transform
- There are no labeled reference cells at all, since the training stage requires cross-entropy
  training on labeled cell types
- The goal is generic cell-type annotation only with no interest in novel rare cell
  discovery — the article notes prior auto-annotation tools focused on annotation
  and paid little attention to novel cell detection; a plain annotation tool would
  be simpler
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scNovel
  role: Neural network framework trained with cross-entropy loss on the labeled reference
    set to output cell-type probability predictions; installed via pip (scNovel==1.1.0)
    and invoked as sr.scNovel(test_adata, train_adata, train_label, processing_unit)
  repo: github.com/chuanyang-Zheng/scNovel
- name: SCANPY
  role: Preprocessing of train/test expression matrices with sc.pp.normalize_total(target_sum=1e4)
    and sc.pp.log1p before training; also used for Leiden clustering and UMAP of detected
    novel cells
- name: PyTorch
  role: Deep-learning backend for the scNovel network, recommended with Cudatoolkit
    for GPU training
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:0f8d0cb53bcb66b32dab9af61e380583ccbc021664c04c5ea235400bdc6228a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - Python
  - SCANPY
  - scNovel
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# neural-network-classifier-training

## Summary

Train scNovel's deep neural network on labeled reference single-cell RNA-seq data with cross-entropy loss so the classifier outputs per-cell-type probability predictions that later feed the query-stage novelty scoring.

## When to use

Use when a labeled reference scRNA-seq matrix and an unlabeled query matrix must be processed through scNovel's two-stage pipeline, i.e. a trained cell-type probability classifier is needed before novel cell detection with threshold lambda = 0.5.

## When NOT to use

- Input is already normalized/log-transformed feature embeddings rather than raw count matrices — reapplying normalize_total(target_sum=1e4) and log1p would double-transform
- There are no labeled reference cells at all, since the training stage requires cross-entropy training on labeled cell types
- The goal is generic cell-type annotation only with no interest in novel rare cell discovery — the article notes prior auto-annotation tools focused on annotation and paid little attention to novel cell detection; a plain annotation tool would be simpler

## Inputs

- Reference scRNA-seq expression matrix with cell-type labels (CSV counts like Experiments/darmanis_counts.csv plus darmanis_label.csv, or AnnData)
- Query scRNA-seq expression matrix to be annotated (e.g. GSE171524 COVID-19 query, 116,314 cells)
- processing_unit setting: 'cpu' (default) or 'gpu'

## Outputs

- Trained scNovel neural network classifier producing per-cell-type probability predictions
- Per-cell predicted labels and novelty scores for thresholding at lambda = 0.5

## How to apply

Load the reference and query expression matrices (e.g. CSV counts or AnnData) and preprocess both identically in Scanpy with sc.pp.normalize_total(target_sum=1e4) followed by sc.pp.log1p. Supply the reference expression matrix together with its label vector (pandas structure) to scNovel's training stage, which fits the neural network using cross-entropy loss to output cell-type probability predictions after the classifier. Choose the processing unit ('cpu' by default, 'gpu' recommended if available) since the model scales to million-level atlases. Keep the two-stage design in mind: the trained classifier is then queried with a novelty threshold lambda = 0.5, and the query stage option chosen (parallel vs sequential) determines which of the four barcode-preprocessing detection score modes (S^MSP, S^ODIN, S^SEQ, S^SIM) is applied downstream. Correctness can be checked against the paper's benchmark: with C^novel = 1 held-out novel class, detection must reach 96.33% AUROC, 99.76% AUPR and 22.82% FPR95.

## Related tools

- **scNovel** (Neural network framework trained with cross-entropy loss on the labeled reference set to output cell-type probability predictions; installed via pip (scNovel==1.1.0) and invoked as sr.scNovel(test_adata, train_adata, train_label, processing_unit)) — github.com/chuanyang-Zheng/scNovel
- **SCANPY** (Preprocessing of train/test expression matrices with sc.pp.normalize_total(target_sum=1e4) and sc.pp.log1p before training; also used for Leiden clustering and UMAP of detected novel cells)
- **PyTorch** (Deep-learning backend for the scNovel network, recommended with Cudatoolkit for GPU training)

## Examples

```
import scNovel as sr; pred_result = sr.scNovel(test_adata, train_adata, train_label, processing_unit='gpu')
```

## Evaluation signals

- With C^novel = 1 held-out novel class, the benchmark run reproduces 96.33% AUROC, 99.76% AUPR and 22.82% FPR95 (Appendix Table 4 / Figure 3A)
- S^SIM attains the highest averaged AUROC among the four score modes S^MSP, S^ODIN, S^SEQ, S^SIM
- Averaged AUROC of scNovel across benchmarks exceeds 94%, up to 16.26% higher than the second-best method
- Both train and test matrices went through the identical Scanpy preprocessing chain (normalize_total target_sum=1e4, then log1p)
- On GSE171524, the trained model's downstream detection rarely flags B cell and T cell as novel while successfully detecting AT1 and AT2 as novel cells

## Limitations

- The score function could be improved — the authors acknowledge 'a better score function could improve scNovel performance'
- The current method does not incorporate other biology information such as single-cell chromatin accessibility data
- Imbalanced clinical data (e.g. the 1.5-million-scale COVID-19 reference vs 116,314-cell GSE171524 query) is the target regime, so training requires large labeled reference data and GPU acceleration is recommended

## Evidence

- [intro] Two-stage training/query design with cross-entropy loss: "During the training stage, we adopt cross-entropy loss to train the model to output cell types’ probability prediction after the classifier."
- [readme] Scanpy preprocessing chain: "sc.pp.normalize_total(train_adata, target_sum=1e4)
sc.pp.log1p(train_adata)"
- [readme] scNovel call signature and processing unit: "pred_result = sr.scNovel(test_adata, train_adata, train_label, processing_unit)"
- [intro] Novelty threshold lambda: [evidence span withheld — anchor status: not_found]
- [abstract] Scalability to million-level atlases: "We validate scNovel's performance on a million-scale dataset to illustrate the scalability of scNovel further."
