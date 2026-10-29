---
name: held-out-cell-validation-model-selection
description: Use when training a custom cTP-net model on paired scRNA-seq/CITE-seq
  data and you must pick optimal network weights via held-out cells rather than training
  loss.
when_to_use_negative:
- Input scRNA-seq is not UMI-based count data — cTP-net can only apply to count-based
  expression input (UMI-based), and TPM/RPKM accuracy is untested
- Data have not been SAVER-X denoised and LogNormalize-transformed — skipping denoising
  retracts prediction correlation by 0.02
- The goal is out-of-cell-type evaluation — that requires partitioning cells by cell
  type and iteratively holding out all cells of one type, not a random 10% split
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: cTP-net
  role: Deep neural network framework whose MB-DNN is trained and whose weights are
    selected on the held-out cells
  repo: https://github.com/zhouzilu/cTPnet
- name: Python
  role: Language in which the shared training script constructs and trains the multiple-branch
    network (Keras/Adam)
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Produces the denoised scRNA-seq input matrix used for training/validation
    before splitting
- name: Seurat
  role: Provides the default LogNormalize transformation applied to denoised counts
    before training
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:72b8e19e84e6a84cb09786e24544341fb231c4f7e82eaa851335e25d622fb4f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - Python
  - SAVER-X
  - Seurat
  - cTP-net
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# Held-out cell validation and model selection for cTP-net training

## Summary

Randomly reserves 10% of paired transcriptome–protein cells as a testing set, trains the MB-DNN on the remaining 90%, and selects the final cTP-net weights (ctpnet_trained_model.h5) by minimizing testing error. This guards the transcriptome-to-protein model against overfitting before deployment on new scRNA-seq data.

## When to use

Use when training a custom cTP-net model on paired scRNA-seq/CITE-seq data and you must pick optimal network weights via held-out cells rather than training loss.

## When NOT to use

- Input scRNA-seq is not UMI-based count data — cTP-net can only apply to count-based expression input (UMI-based), and TPM/RPKM accuracy is untested
- Data have not been SAVER-X denoised and LogNormalize-transformed — skipping denoising retracts prediction correlation by 0.02
- The goal is out-of-cell-type evaluation — that requires partitioning cells by cell type and iteratively holding out all cells of one type, not a random 10% split

## Inputs

- SAVER-X denoised, Seurat LogNormalize-normalized scRNA-seq count matrix X (genes x cells)
- Relative-abundance (geometric mean log-ratio) transformed surface protein ADT counts from paired CITE-seq/REAP-seq cells
- Training script training_05152020.py from the cTPnet repository

## Outputs

- Selected model weights saved as ctpnet_trained_model.h5
- Testing error on the 10% held-out cells used for model selection

## How to apply

Obtain the shared training script training_05152020.py from github.com/zhouzilu/cTPnet. Load the SAVER-X denoised, LogNormalize-normalized scRNA-seq matrix X as input and the relative-abundance-transformed (geometric mean log-ratio) surface protein counts as the response. Randomly split cells into 90% training and 10% testing, then train the multiple-branch network — a D-gene input feeding a 1000-node shared layer, a 128-node bottleneck, then per-protein 64-node ReLU branches with identity outputs — with an L1 objective optimized by Adam at learning rate 10e-5 for 139 epochs. Select the checkpoint with the lowest error on the 10% held-out cells and save those weights as ctpnet_trained_model.h5. Rationale: random cell-level holdout estimates generalization to new cells while keeping most paired data for fitting; the article explicitly states 'The optimal model was selected based on the testing error.'

## Related tools

- **cTP-net** (Deep neural network framework whose MB-DNN is trained and whose weights are selected on the held-out cells) — https://github.com/zhouzilu/cTPnet
- **Python** (Language in which the shared training script constructs and trains the multiple-branch network (Keras/Adam)) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Produces the denoised scRNA-seq input matrix used for training/validation before splitting)
- **Seurat** (Provides the default LogNormalize transformation applied to denoised counts before training)

## Examples

```
python training_05152020.py  # trains the MB-DNN on 90% of paired cells, selects weights by 10% testing error, saves ctpnet_trained_model.h5
```

## Evaluation signals

- Model weights are saved from the epoch/checkpoint with the lowest testing error on the 10% held-out cells, not simply the final epoch
- Output file ctpnet_trained_model.h5 exists and is loadable by the cTPnet package
- Training used the specified architecture (1000-node shared layer, 128-node bottleneck, 64-node per-protein ReLU branches) with Adam at learning rate 10e-5 for 139 epochs and an L1 objective
- Prediction correlation on held-out cells should degrade by about 0.02 if SAVER-X denoising is omitted — a sanity check that inputs were correctly preprocessed
- Held-out predictions of relative protein abundance should outperform those of a network trained on raw protein barcode counts

## Limitations

- With the random 10% holdout, comparisons across methods must be limited to the same cells in the validation set to account for variations between subsets
- A random split does not test generalization to unseen cell types; for that, partition cells by cell type and hold out all cells of one type
- Predictions were only benchmarked on hematopoietic (PBMC/CBMC/BMMC) cells; generalization outside this lineage was not tested
- Training your own model is for research use — commercial usage requires contacting the authors (Zilu Zhou or Nancy Zhang)

## Evidence

- [discussion] Validation split 90/10 with model selection on testing error: "we keep only 10% of the cells as the testing set, and use the other 90% of the cells for training. The optimal model was selected based on the testing error."
- [methods] Shared training script location: "we have decided to share our training python code [***here***](https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py)"
- [discussion] Optimization settings for training: "The objective function was optimized stochastically with Adam46 with learning rate set to 10e–5 for 139 epochs"
- [discussion] Relative abundance response improves accuracy: "The network trained using this transformed relative protein abundance as the response vector yields better prediction accuracy than the network trained using raw protein barcode counts."
- [discussion] Out-of-cell-type validation is a different holdout scheme: "instead of selecting the test set cells randomly, we partition the cells by their cell types"
