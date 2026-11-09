---
name: novel-cell-detection-mode-ablation
description: Use when benchmarking or configuring scNovel for novel rare cell-type
  discovery and you must choose among its four detection score modes (S^MSP, S^ODIN,
  S^SEQ, S^SIM) using AUROC/AUPR/FPR95.
when_to_use_negative:
- The task is generic cell-type annotation of query cells matching known reference
  types, not detecting novel/rare cell types absent from the reference
- You need only run scNovel's default annotation pipeline rather than compare or tune
  its four detection score modes
- Your goal is comparing scNovel against other novel cell detection methods (e.g.
  scLearn, scPred) rather than ablating scNovel's internal score modes
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scNovel
  role: Deep neural network classifier and source of the four novel cell detection
    score modes (S^MSP, S^ODIN, S^SEQ, S^SIM) under ablation
  repo: github.com/chuanyang-Zheng/scNovel
- name: SCANPY
  role: Loading expression matrices and applying sc.pp.normalize_total(target_sum=1e4)
    and sc.pp.log1p preprocessing to both reference and query sets
- name: PyTorch
  role: Backend for training the scNovel classifier with cross-entropy loss
- name: Python
  role: Language environment for running the ablation pipeline (scNovel is an open-source
    Python package)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:0e754250cffae4e34bb712d7c40b1c1c0beaebf2b1b4877c68a4cf0723cd7371
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - Python
  - SCANPY
  - scNovel
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# novell-cell-detection-mode-ablation

## Summary

An ablation protocol for scNovel's four novel cell detection score modes — S^MSP (maximum prediction probability), S^ODIN, S^SEQ (sequential barcode preprocessing), and S^SIM (simultaneous barcode preprocessing) — to determine which mode best detects novel rare cell types in scRNA-seq data.

## When to use

Use when benchmarking or configuring scNovel for novel rare cell-type discovery and you must choose among its four detection score modes (S^MSP, S^ODIN, S^SEQ, S^SIM) using AUROC/AUPR/FPR95.

## When NOT to use

- The task is generic cell-type annotation of query cells matching known reference types, not detecting novel/rare cell types absent from the reference
- You need only run scNovel's default annotation pipeline rather than compare or tune its four detection score modes
- Your goal is comparing scNovel against other novel cell detection methods (e.g. scLearn, scPred) rather than ablating scNovel's internal score modes

## Inputs

- Reference scRNA-seq expression matrix with cell-type labels (CSV, e.g. darmanis_counts.csv / darmanis_label.csv format)
- Query scRNA-seq expression matrix containing held-out novel cell classes (C^novel = 1–4)
- Cell-type label vectors in pandas structure for the reference set

## Outputs

- Mode-comparison table reproducing Appendix Table 4 with AUROC, AUPR, and FPR95 per mode (S^MSP, S^ODIN, S^SEQ, S^SIM)
- Figure 3A-style comparison plot of the four detection modes
- Selected best mode (expected: S^SIM) with averaged AUROC across C^novel settings

## How to apply

Load reference and query expression matrices with scanpy, applying sc.pp.normalize_total(target_sum=1e4) followed by sc.pp.log1p to both sets. Train the scNovel deep neural network classifier on reference cell types with cross-entropy loss using the two-stage design with a novelty threshold lambda = 0.5. Instantiate the four detection scores: S^MSP (Equation C6, maximum prediction probability), S^ODIN (Equation C10, T_minus barcode preprocessing only), S^SEQ (Equation C13, sequential T_minus then T_plus preprocessing), and S^SIM (Equation C12, simultaneous T_minus and T_plus preprocessing). Evaluate each mode on benchmark datasets with C^novel = 1, 2, 3, and 4 held-out novel classes, recording AUROC as the main metric plus AUPR and FPR95. Compile a mode-comparison table reproducing Appendix Table 4 and a Figure 3A-style comparison; the ablation concludes S^SIM is the best mode for novel rare cell-type discovery.

## Related tools

- **scNovel** (Deep neural network classifier and source of the four novel cell detection score modes (S^MSP, S^ODIN, S^SEQ, S^SIM) under ablation) — github.com/chuanyang-Zheng/scNovel
- **SCANPY** (Loading expression matrices and applying sc.pp.normalize_total(target_sum=1e4) and sc.pp.log1p preprocessing to both reference and query sets)
- **PyTorch** (Backend for training the scNovel classifier with cross-entropy loss)
- **Python** (Language environment for running the ablation pipeline (scNovel is an open-source Python package))

## Examples

```
import scanpy as sc; sc.pp.normalize_total(train_adata, target_sum=1e4); sc.pp.log1p(train_adata); pred_result = sr.scNovel(test_adata, train_adata, train_label, 'gpu')
```

## Evaluation signals

- The C^novel=1 run must reproduce 96.33% AUROC, 99.76% AUPR, and 22.82% FPR95
- S^SIM must attain the highest averaged AUROC among the four modes, confirming the Appendix Table 4 / Figure 3A conclusion
- All modes are evaluated under identical preprocessing: sc.pp.normalize_total(target_sum=1e4) then sc.pp.log1p on both reference and query matrices
- AUPR and FPR95 are reported alongside AUROC for each mode and each C^novel setting (1, 2, 3, 4)
- The trained classifier uses cross-entropy loss with the two-stage design and novelty threshold lambda = 0.5

## Limitations

- The article acknowledges that a better score function could improve scNovel performance, so mode rankings may shift with improved score designs
- Results depend on the two-stage design and threshold lambda = 0.5; changing these may invalidate the mode comparison
- The benchmark comparison is valid for C^novel = 1–4 held-out novel classes; behavior outside this range is not characterized here

## Evidence

- [intro] Four novel cell detection modes determined by the selected query stage option: "barcode preprocessing is utilized to boost model performance via four modes of novel cell detection"
- [intro] Two-stage usage of scNovel: "During the training stage, we adopt cross-entropy loss to train the model to output cell types’ probability prediction after the classifier. During the query stage"
- [readme] Package dependencies: "- Scanpy (Compatible with all versions)
- Pytorch (With Cudatoolkit is recommended)"
- [readme] Preprocessing steps: "sc.pp.normalize_total(train_adata, target_sum=1e4)
sc.pp.log1p(train_adata)"
- [readme] Core scNovel call: "pred_result = sr.scNovel(test_adata, train_adata, train_label, processing_unit)"
