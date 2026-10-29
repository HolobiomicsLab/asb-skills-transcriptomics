---
name: multi-branch-network-architecture-implementation
description: Use when you have paired scRNA-seq and CITE-seq/REAP-seq surface protein
  measurements and need to train a transcriptome-to-protein mapping model rather than
  apply the pre-trained cTP-net model.
when_to_use_negative:
- 'Input expression is TPM/RPKM or otherwise non-UMI: ''cTP-net can only apply to
  count-based expression input (UMI-based)'''
- You only want to impute surface proteins with the published model — use the cTPnet
  R/Python package inference pipeline instead of retraining
- Target cell types are outside the hematopoietic lineage, which was not tested for
  generalization
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: cTP-net
  role: Framework whose multiple-branch deep neural network architecture is being
    implemented and trained
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoises the scRNA-seq training input prior to network training
- name: Seurat
  role: LogNormalize transformation of the denoised counts used as network input
- name: Python
  role: Language in which the MB-DNN training script (training_05152020.py) is written
  repo: https://github.com/zhouzilu/cTPnet
- name: Adam (optimizer)
  role: Stochastic optimizer for the L1 objective, learning rate 1e-5, 139 epochs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:b49a75f2e0d1057aaba5527aaf1bc9ad2e413bad8dfc48674af214de1fa222fa
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
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# Multi-branch network architecture implementation

## Summary

Implement and train cTP-net's multiple-branch deep neural network (MB-DNN) that maps a cell's SAVER-X denoised, LogNormalize-normalized transcriptome to relative abundances of surface proteins, with a 128-node bottleneck feeding protein-specific 64-node branches. The shared training script (training_05152020.py) enables retraining on new paired transcriptome-protein datasets.

## When to use

Use when you have paired scRNA-seq and CITE-seq/REAP-seq surface protein measurements and need to train a transcriptome-to-protein mapping model rather than apply the pre-trained cTP-net model.

## When NOT to use

- Input expression is TPM/RPKM or otherwise non-UMI: 'cTP-net can only apply to count-based expression input (UMI-based)'
- You only want to impute surface proteins with the published model — use the cTPnet R/Python package inference pipeline instead of retraining
- Target cell types are outside the hematopoietic lineage, which was not tested for generalization

## Inputs

- SAVER-X denoised, LogNormalize-normalized scRNA-seq count matrix X (UMI-based counts)
- Relative-abundance-transformed surface protein ADT counts (paired CITE-seq/REAP-seq response vector)

## Outputs

- Trained MB-DNN model weights (e.g., ctpnet_trained_model.h5) mapping transcriptome to per-protein relative abundance
- Held-out testing-set prediction error used for model selection

## How to apply

Obtain the shared training script training_05152020.py from github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py. Prepare inputs: SAVER-X denoised scRNA-seq counts (Λ) transformed with Seurat default LogNormalize as matrix X, and surface protein counts converted by the relative abundance (geometric mean log-ratio) transformation from Stoeckius et al. as the response vector — this transformed response yields better prediction accuracy than raw protein barcode counts. Construct the network: D-gene input layer → 1000-node shared layer → 128-node bottleneck → per-protein 64-node branches with ReLU activations and identity outputs. Train with an L1 objective optimized by Adam (learning rate 1e-5) for 139 epochs, keeping 10% of cells as testing set (90% training) and selecting the optimal model by testing error before saving weights (e.g., ctpnet_trained_model.h5).

## Related tools

- **cTP-net** (Framework whose multiple-branch deep neural network architecture is being implemented and trained) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoises the scRNA-seq training input prior to network training)
- **Seurat** (LogNormalize transformation of the denoised counts used as network input)
- **Python** (Language in which the MB-DNN training script (training_05152020.py) is written) — https://github.com/zhouzilu/cTPnet
- **Adam (optimizer)** (Stochastic optimizer for the L1 objective, learning rate 1e-5, 139 epochs)

## Examples

```
git clone https://github.com/zhouzilu/cTPnet; python cTPnet/extdata/training_05152020.py  # train MB-DNN on paired SAVER-X/LogNormalize RNA and relative-abundance protein counts
```

## Evaluation signals

- Model selection criterion: lowest prediction error on the 10% held-out testing cells
- Predicted relative abundances correlate better with measured CITE-seq abundances than raw RNA counts or SAVER-X denoised RNA alone (per Fig. 1b comparisons)
- Removing SAVER-X denoising should degrade correlation by ~0.02; if your trained model matches that sensitivity, the input pipeline is correct
- Architecture invariants: 1000-node shared layer, 128-node bottleneck, 64-node ReLU branches per protein with identity outputs
- Training converges under Adam lr 1e-5 within 139 epochs

## Limitations

- without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation
- cTP-net can only apply to count-based expression input (UMI-based)
- A model extending prediction to 100 surface proteins is currently in training; the released model covers a smaller panel (e.g., 24 immunophenotype markers)
- Generalization to cell types outside the hematopoietic lineage was not tested

## Evidence

- [discussion] The objective function was optimized stochastically with Adam46 with learning rate set to 10e–5 for 139 epochs: "The objective function was optimized stochastically with Adam46 with learning rate set to 10e–5 for 139 epochs"
- [discussion] 90/10 split and model selection: "we keep only 10% of the cells as the testing set, and use the other 90% of the cells for training. The optimal model was selected based on the testing error."
- [discussion] Training input definition: "The normalized denoised count matrix X is the training input for the subsequent multiple branch neural network."
- [readme] Shared training script availability: "we have decided to share our training python code [***here***](https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py)"
- [discussion] Response transformation rationale: "The network trained using this transformed relative protein abundance as the response vector yields better prediction accuracy than the network trained using raw protein barcode counts."
