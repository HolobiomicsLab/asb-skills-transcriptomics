---
name: out-of-distribution-score-function-implementation
description: Use when a trained cell-type classifier must flag query cells from unseen
  cell types, and you need to choose among maximum-probability, ODIN, or barcode-preprocessing
  (sequential/simultaneous) scoring modes.
when_to_use_negative:
- The task is generic cell-type annotation only, with no need to identify novel cell
  types absent from the reference — scNovel's score modes target novel rare cell detection
  specifically
- You cannot hold out known cell types to form a labeled novel-class benchmark (C^novel
  classes with AUROC/AUPR/FPR95 ground truth), since mode selection requires this
  evaluation setup
- Input data are already normalized to a different scale or not count matrices compatible
  with sc.pp.normalize_total(target_sum=1e4) + sc.pp.log1p preprocessing
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scNovel
  role: Framework providing the two-stage neural network (cross-entropy training,
    lambda-thresholded query stage) and the barcode-preprocessing score modes S^MSP,
    S^ODIN, S^SEQ, S^SIM
  repo: github.com/chuanyang-Zheng/scNovel
- name: SCANPY
  role: Preprocessing of reference and query expression matrices via sc.pp.normalize_total(target_sum=1e4)
    and sc.pp.log1p before scoring
- name: PyTorch
  role: Deep-learning backend for training the scNovel classifier whose prediction
    probabilities feed the score functions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae112
    title: 'scNovel: a scalable deep learning-based network for novel rare cell discovery
      in single-cell transcriptomics'
  source_content_hash: sha256:8ccd590ceb19485fc1955c6998d2ac51772727309d5fff3522473011786406a3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - Python
  - SCANPY
  - scNovel
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# out-of-distribution-score-function-implementation

## Summary

Implements the four novel-rare-cell detection score modes of scNovel (S^MSP maximum prediction probability, S^ODIN, S^SEQ sequential, and S^SIM simultaneous barcode preprocessing) on top of a trained neural network classifier, and benchmarks them with AUROC/AUPR/FPR95 to select the best mode for novel rare cell-type discovery.

## When to use

Use when a trained cell-type classifier must flag query cells from unseen cell types, and you need to choose among maximum-probability, ODIN, or barcode-preprocessing (sequential/simultaneous) scoring modes.

## When NOT to use

- The task is generic cell-type annotation only, with no need to identify novel cell types absent from the reference — scNovel's score modes target novel rare cell detection specifically
- You cannot hold out known cell types to form a labeled novel-class benchmark (C^novel classes with AUROC/AUPR/FPR95 ground truth), since mode selection requires this evaluation setup
- Input data are already normalized to a different scale or not count matrices compatible with sc.pp.normalize_total(target_sum=1e4) + sc.pp.log1p preprocessing

## Inputs

- Reference scRNA-seq expression matrix with cell-type label vector (e.g. train_adata.csv and train_label.csv as in the scNovel tutorial)
- Query/test scRNA-seq expression matrix (e.g. test_adata.csv)
- Trained scNovel neural network classifier (two-stage design, novelty threshold lambda = 0.5)

## Outputs

- Per-cell out-of-distribution scores under each of the four modes (S^MSP, S^ODIN, S^SEQ, S^SIM)
- Mode-comparison table with AUROC (main), AUPR, and FPR95 across C^novel = 1, 2, 3, 4 held-out novel classes (Appendix Table 4 style)
- Figure 3A-style comparison confirming the best-performing mode

## How to apply

Load the reference and query scRNA-seq matrices with scanpy, applying sc.pp.normalize_total(target_sum=1e4) followed by sc.pp.log1p to both sets. Train the scNovel deep-neural-network classifier on reference cell types with cross-entropy loss using the two-stage design (training stage, then query stage with novelty threshold lambda = 0.5). Instantiate the four detection scores: S^MSP (Equation C6, maximum prediction probability), S^ODIN (Equation C10, T_minus barcode preprocessing only), S^SEQ (Equation C13, sequential T_minus then T_plus preprocessing), and S^SIM (Equation C12, simultaneous T_minus and T_plus preprocessing), where the two-stage design supports parallel and sequential query-stage options. Evaluate each mode on benchmark datasets with C^novel = 1, 2, 3, and 4 held-out novel classes, recording AUROC as the main metric plus AUPR and FPR95 for reference, then compile a mode-comparison table in the style of Appendix Table 4 and Figure 3A. Decision rule: pick the mode with the highest averaged AUROC across the C^novel settings — the ablation concludes S^SIM is the best mode for novel rare cell-type discovery.

## Related tools

- **scNovel** (Framework providing the two-stage neural network (cross-entropy training, lambda-thresholded query stage) and the barcode-preprocessing score modes S^MSP, S^ODIN, S^SEQ, S^SIM) — github.com/chuanyang-Zheng/scNovel
- **SCANPY** (Preprocessing of reference and query expression matrices via sc.pp.normalize_total(target_sum=1e4) and sc.pp.log1p before scoring)
- **PyTorch** (Deep-learning backend for training the scNovel classifier whose prediction probabilities feed the score functions)

## Examples

```
import scNovel as sr; pred_result = sr.scNovel(test_adata, train_adata, train_label, 'gpu')
```

## Evaluation signals

- The C^novel = 1 run must reproduce 96.33% AUROC, 99.76% AUPR, and 22.82% FPR95
- S^SIM must attain the highest averaged AUROC among the four modes across C^novel = 1, 2, 3, 4 (Appendix Table 4 / Figure 3A pattern)
- Both reference and query sets show identical preprocessing (normalize_total target_sum=1e4 then log1p) before scoring
- All four modes report the full metric triple (AUROC, AUPR, FPR95) rather than AUROC alone
- Novel-class detection rates should rise consistently as C^novel increases from 1 to 4 held-out classes

## Limitations

- The authors acknowledge the score function could be improved — a better-designed score function could improve scNovel performance
- The current method does not incorporate other biology information such as single-cell chromatin accessibility sequencing data
- No version numbers are given for scNovel or Python in the discussion; the README pins scNovel==1.1.0 with Scanpy, PyTorch, Numpy > 1.20, Pandas > 1.2

## Evidence

- [intro] Four novel cell detection modes determined by the selected query stage option: "barcode preprocessing is utilized to boost model performance via four modes of novel cell detection"
- [intro] Two query-stage options underpin the score modes: "We develop two optional query stage options: the parallel query stage and the sequential query stage"
- [readme] Score functions drive the outperformance on novel rare cell detection: "By leveraging the newly designed neural network structure and proposed two new score functions, scRare especially obtains an outperformance on novel rare cell detection"
- [readme] Preprocessing requirement for score computation: "sc.pp.normalize_total(train_adata, target_sum=1e4)
sc.pp.log1p(train_adata)"
