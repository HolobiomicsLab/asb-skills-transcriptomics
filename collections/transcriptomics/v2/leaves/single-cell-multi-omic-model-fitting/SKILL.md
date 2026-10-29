---
name: single-cell-multi-omic-model-fitting
description: Use when paired transcriptome and surface protein (CITE-seq/REAP-seq
  ADT) data are available for training and the goal is to impute relative surface
  protein abundances from UMI-based scRNA-seq counts alone.
when_to_use_negative:
- Input expression data is not UMI-based count data (e.g. TPM/RPKM) — cTP-net can
  only apply to count-based UMI expression input, and prediction accuracy on TPM/RPKM
  is unknown.
- You need imputation of non-hematopoietic or out-of-panel proteins — generalization
  outside the hematopoietic lineage was not tested, and the 100-protein model was
  still in training.
- You only need cell-type clustering/annotation from transcriptome alone with no protein
  imputation target.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: cTP-net
  role: Imputes surface protein abundance from denoised scRNA-seq data; MB-DNN model
    training and prediction
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoises scRNA-seq counts before imputation; omitting it retracts correlation
    performance by 0.02
- name: Seurat
  role: Default LogNormalize transformation of SAVER-X denoised counts (Λ)
- name: Python
  role: Training script language (training_05152020.py) for constructing and fitting
    the MB-DNN with Adam
  repo: https://github.com/zhouzilu/cTPnet
- name: Adam (optimizer)
  role: Stochastic optimizer of the L1 objective, learning rate 10e-5 for 139 epochs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:993a93a50a2bbb4ea0ca5e2aa7a5ddaae002ced6d8969bc7d5153cd418629142
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
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# single-cell-multi-omic-model-fitting

## Summary

Fit a multiple-branch deep neural network (MB-DNN) that maps a SAVER-X-denoised, LogNormalize-normalized scRNA-seq count matrix to the geometric-mean log-ratio (relative abundance) transformed surface protein counts, imputing protein abundances for single cells.

## When to use

Use when paired transcriptome and surface protein (CITE-seq/REAP-seq ADT) data are available for training and the goal is to impute relative surface protein abundances from UMI-based scRNA-seq counts alone.

## When NOT to use

- Input expression data is not UMI-based count data (e.g. TPM/RPKM) — cTP-net can only apply to count-based UMI expression input, and prediction accuracy on TPM/RPKM is unknown.
- You need imputation of non-hematopoietic or out-of-panel proteins — generalization outside the hematopoietic lineage was not tested, and the 100-protein model was still in training.
- You only need cell-type clustering/annotation from transcriptome alone with no protein imputation target.

## Inputs

- SAVER-X denoised, Seurat LogNormalize-normalized scRNA-seq UMI count matrix X (genes x cells)
- Relative-abundance (geometric mean log-ratio) transformed surface protein ADT counts as response vector
- Gene names for the 24 immunophenotype marker panel (e.g. gene_names_protein12)

## Outputs

- Trained model weights (e.g. ctpnet_trained_model.h5)
- Imputed relative surface protein abundances per cell for the protein panel

## How to apply

Preprocess the scRNA-seq input by denoising with SAVER-X default parameters and applying Seurat's default LogNormalize transformation; remove low-quality genes (<10 counts across cells) and low-quality cells (<200 genes detected). Transform the surface protein ADT counts with the Stoeckius et al. relative abundance (geometric mean log-ratio) transformation, since training on transformed relative abundance outperforms raw barcode counts. Construct the MB-DNN (e.g. using the shared script training_05152020.py from github.com/zhouzilu/cTPnet): an input layer of D genes feeding a 1000-node shared layer, a 128-node bottleneck, then per-protein 64-node branches with ReLU activations and identity outputs. Train with an L1 objective optimized by Adam (learning rate 10e-5) for 139 epochs, keeping 90% of cells for training and 10% as a testing set for model selection; save the selected weights (e.g. ctpnet_trained_model.h5). For out-of-cell-type evaluation, partition cells by cell type and iteratively hold out all cells of one type for testing.

## Related tools

- **cTP-net** (Imputes surface protein abundance from denoised scRNA-seq data; MB-DNN model training and prediction) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoises scRNA-seq counts before imputation; omitting it retracts correlation performance by 0.02)
- **Seurat** (Default LogNormalize transformation of SAVER-X denoised counts (Λ))
- **Python** (Training script language (training_05152020.py) for constructing and fitting the MB-DNN with Adam) — https://github.com/zhouzilu/cTPnet
- **Adam (optimizer)** (Stochastic optimizer of the L1 objective, learning rate 10e-5 for 139 epochs)

## Examples

```
git clone https://github.com/zhouzilu/cTPnet && python extdata/training_05152020.py
```

## Evaluation signals

- Model selection uses testing error on the 10% held-out cells; the saved model should be the lowest testing-error epoch.
- Predicted vs CITE-seq measured protein abundance should show high cross-cell correlation; omitting SAVER-X denoising retracts correlation by 0.02, so correlated performance below that signals a preprocessing error.
- Out-of-cell-type evaluation should hold out ALL cells of one type (not random cells) and still show good generalization to related cell types.
- Feature-space alignment check: genes in training but not testing data must be filled with zeros so input dimension D matches the trained model.
- Training on raw protein barcode counts instead of the transformed relative abundance should yield worse prediction accuracy — verify the response was geometric-mean log-ratio transformed.

## Limitations

- cTP-net can only apply to count-based expression input (UMI-based); TPM/RPKM accuracy is untested.
- Generalization to cell types outside the hematopoietic lineage has not been tried.
- Windows is not recommended (ctpnet Python module import issues; numpy 1.19.3 required); no GPU required — a 4000-cell run takes <1 min.
- The 100-surface-protein model was still in training; the released model covers a smaller marker panel (e.g. 24 immunophenotype markers).

## Evidence

- [discussion] Network architecture and objective: "The normalized denoised count matrix X is the training input for the subsequent multiple branch neural network."
- [discussion] Train/test split: "we keep only 10% of the cells as the testing set, and use the other 90% of the cells for training. The optimal model was selected based on the testing error."
- [discussion] Optimizer settings: "The objective function was optimized stochastically with Adam46 with learning rate set to 10e–5 for 139 epochs"
- [methods] Training data location: "we have decided to share our training python code [***here***](https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py)"
- [discussion] Response transformation: "The network trained using this transformed relative protein abundance as the response vector yields better prediction accuracy than the network trained using raw protein barcode counts."
- [discussion] SAVER-X contribution: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
