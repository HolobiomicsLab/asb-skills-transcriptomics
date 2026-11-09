---
name: l1-loss-regression-optimization
description: Use when training a cTP-net MB-DNN to map a LogNormalize-normalized,
  SAVER-X-denoised scRNA-seq matrix to relative-abundance-transformed surface protein
  counts and a per-protein regression loss must be selected.
when_to_use_negative:
- Input expression is TPM/RPKM or otherwise not UMI-based count data — the article
  states cTP-net can only apply to count-based (UMI-based) expression input and accuracy
  for TPM/RPKM CITE-seq is unknown
- scRNA-seq data has not been denoised — omitting SAVER-X denoising retracts prediction
  correlation by 0.02, more than any other parameter tweak
- The target cells are not of the hematopoietic lineage — generalization outside hematopoietic
  cells was not tested
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: cTP-net
  role: MB-DNN framework whose L1 objective is optimized by Adam to impute surface
    protein relative abundances from scRNA-seq data
  repo: https://github.com/zhouzilu/cTPnet
- name: Adam (optimizer)
  role: Stochastic optimizer of the L1 objective with learning rate 10e-5 for 139
    epochs
- name: SAVER-X
  role: Denoises the scRNA-seq training input before MB-DNN training; strongly recommended
    preprocessing
- name: Seurat
  role: LogNormalize transformation of SAVER-X denoised counts to produce the network
    input matrix
- name: Python
  role: Language in which the MB-DNN is constructed and the L1 training is performed
    via the shared training script
  repo: https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:181bffdc7394320dc4d790dd0b74bb00ca0f1d77b704928880bfba056b84a4d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - Adam (optimizer)
  - Python
  - SAVER-X
  - Seurat
  - cTP-net
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# L1-loss regression optimization

## Summary

Training recipe for the cTP-net multiple-branch deep neural network (MB-DNN) in which an L1 objective is stochastically optimized with Adam to learn the transcriptome-to-relative-surface-protein-abundance mapping.

## When to use

Use when training a cTP-net MB-DNN to map a LogNormalize-normalized, SAVER-X-denoised scRNA-seq matrix to relative-abundance-transformed surface protein counts and a per-protein regression loss must be selected.

## When NOT to use

- Input expression is TPM/RPKM or otherwise not UMI-based count data — the article states cTP-net can only apply to count-based (UMI-based) expression input and accuracy for TPM/RPKM CITE-seq is unknown
- scRNA-seq data has not been denoised — omitting SAVER-X denoising retracts prediction correlation by 0.02, more than any other parameter tweak
- The target cells are not of the hematopoietic lineage — generalization outside hematopoietic cells was not tested

## Inputs

- SAVER-X denoised, Seurat LogNormalize-normalized scRNA-seq count matrix X (UMI-based counts, genes x cells)
- Relative-abundance (geometric mean log-ratio) transformed surface protein ADT counts (response vector)
- training_05152020.py shared training script from github.com/zhouzilu/cTPnet

## Outputs

- Trained cTP-net MB-DNN model weights (e.g. ctpnet_trained_model.h5)
- Predicted relative abundances of surface proteins per cell
- Testing-error-based model selection trace over 139 epochs

## How to apply

Obtain the shared training script training_05152020.py from the cTPnet repository and load the paired training data: the SAVER-X denoised, LogNormalize-normalized scRNA-seq count matrix X as input and relative-abundance-transformed surface protein counts as the response (training on transformed relative abundance outperforms training on raw barcode counts). Split cells 90/10 into training and testing sets, then construct the multiple-branch network: an input layer of D genes feeding a 1000-node shared layer, a 128-node bottleneck, then per-protein 64-node branches with ReLU activations and identity outputs. Optimize the L1 objective stochastically with Adam at learning rate 10e-5 for 139 epochs, and select the optimal model based on the testing error on the held-out 10% of cells; save selected weights (e.g. as ctpnet_trained_model.h5).

## Related tools

- **cTP-net** (MB-DNN framework whose L1 objective is optimized by Adam to impute surface protein relative abundances from scRNA-seq data) — https://github.com/zhouzilu/cTPnet
- **Adam (optimizer)** (Stochastic optimizer of the L1 objective with learning rate 10e-5 for 139 epochs)
- **SAVER-X** (Denoises the scRNA-seq training input before MB-DNN training; strongly recommended preprocessing)
- **Seurat** (LogNormalize transformation of SAVER-X denoised counts to produce the network input matrix)
- **Python** (Language in which the MB-DNN is constructed and the L1 training is performed via the shared training script) — https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py

## Examples

```
python training_05152020.py  # builds MB-DNN (1000-node shared layer, 128-node bottleneck, 64-node branches), optimizes L1 loss with Adam lr=10e-5 for 139 epochs, selects model on 10% testing set
```

## Evaluation signals

- The optimal model is chosen by testing error on a 10% held-out cell set (90% used for training)
- Network architecture invariants: 1000-node shared layer, 128-node bottleneck, 64-node per-protein branches, ReLU activations, identity outputs
- Training converges within the specified 139 epochs at Adam learning rate 10e-5 under the L1 objective
- Predicted relative protein abundances correlate better with CITE-seq measured abundances than raw RNA counts; training on relative-abundance-transformed response should outperform training on raw protein barcode counts
- Prediction performance should drop ~0.02 in correlation if SAVER-X denoising is omitted from the input pipeline

## Limitations

- cTP-net can only apply to count-based expression input (UMI-based); prediction accuracy for TPM/RPKM CITE-seq data is unknown
- Imputation of immune-related markers on cells not of the hematopoietic lineage has not been attempted
- Benchmark comparisons are limited to the same cells in the validation set to account for variations between subsets
- The shared model covers a set of immunophenotype markers (e.g. 24 markers); a model extending prediction to 100 surface proteins was still in training
- Windows platforms are not recommended for running cTP-net (ctpnet module import issues; numpy 1.19.3 workaround)

## Evidence

- [discussion] The objective function was optimized stochastically with Adam46 with learning rate set to 10e–5 for 139 epochs: "The objective function was optimized stochastically with Adam46 with learning rate set to 10e–5 for 139 epochs"
- [discussion] The normalized denoised count matrix X is the training input: "The normalized denoised count matrix X is the training input for the subsequent multiple branch neural network."
- [discussion] 10% testing set / 90% training set selection: "we keep only 10% of the cells as the testing set, and use the other 90% of the cells for training. The optimal model was selected based on the testing error."
- [discussion] Relative abundance transformation of protein counts yields better training response: "The network trained using this transformed relative protein abundance as the response vector yields better prediction accuracy than the network trained using raw protein barcode counts."
- [readme] README: shared training python code: "we have decided to share our training python code [***here***](https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py)"
- [discussion] cTP-net only applies to UMI-based count input: "cTP-net can only apply to count-based expression input (UMI-based)."
