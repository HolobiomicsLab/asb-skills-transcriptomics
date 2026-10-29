---
name: deep-neural-network-training
description: Use when paired CITE-seq/REAP-seq transcriptome and surface protein data
  are available and a trained model must be produced (or retrained for ablations)
  that imputes protein relative abundances from scRNA-seq counts.
when_to_use_negative:
- Expression data are not UMI-based counts — the paper states cTP-net can only apply
  to count-based expression input (UMI-based), and TPM/RPKM accuracy is unknown
- No paired transcriptome–surface protein training data are available and the goal
  is simply to impute proteins with the pretrained model (use the cTPnet prediction
  package instead of retraining)
- Input scRNA-seq has not been quality-filtered and denoised (genes with <10 counts
  across cells, cells with <200 detected genes removed, SAVER-X denoising)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: cTP-net
  role: The multiple-branch deep neural network being trained to map transcriptome
    to surface protein relative abundance
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoises the scRNA-seq training input before model training; removing it costs
    ~0.02 correlation
- name: Seurat
  role: LogNormalize transformation of the denoised counts to produce the network
    input matrix X
- name: Adam (optimizer)
  role: Stochastic optimizer of the L1 objective at learning rate 10e-5 for 139 epochs
- name: Python
  role: Language in which the MB-DNN is constructed and trained via the shared training
    script
  repo: https://github.com/zhouzilu/cTPnet
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:c863672afaa3528ee7cd616dc87bd2c68a36e53450500ca09cfea932bdff74cb
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# deep-neural-network-training

## Summary

Trains the cTP-net multiple-branch deep neural network (MB-DNN) that maps a SAVER-X-denoised, LogNormalize-normalized scRNA-seq matrix to relative surface protein abundances, using the authors' shared training script and Adam optimization.

## When to use

Use when paired CITE-seq/REAP-seq transcriptome and surface protein data are available and a trained model must be produced (or retrained for ablations) that imputes protein relative abundances from scRNA-seq counts.

## When NOT to use

- Expression data are not UMI-based counts — the paper states cTP-net can only apply to count-based expression input (UMI-based), and TPM/RPKM accuracy is unknown
- No paired transcriptome–surface protein training data are available and the goal is simply to impute proteins with the pretrained model (use the cTPnet prediction package instead of retraining)
- Input scRNA-seq has not been quality-filtered and denoised (genes with <10 counts across cells, cells with <200 detected genes removed, SAVER-X denoising)

## Inputs

- SAVER-X denoised, Seurat LogNormalize-normalized scRNA-seq count matrix X (genes x cells)
- Relative-abundance-transformed (geometric mean log-ratio) surface protein ADT counts from CITE-seq/REAP-seq (response vector)
- Shared training script training_05152020.py from github.com/zhouzilu/cTPnet

## Outputs

- Trained cTP-net model weights saved as ctpnet_trained_model.h5
- Per-protein Pearson correlations between predicted and measured protein relative abundances on the 10% testing cells

## How to apply

Obtain the shared training script training_05152020.py from github.com/zhouzilu/cTPnet and load the paired training data: the SAVER-X denoised, Seurat LogNormalize-normalized scRNA-seq count matrix X as input and the relative-abundance-transformed surface protein counts as the response vector (the paper found this transformed response yields better accuracy than raw ADT barcode counts). Split cells 90%/10% into training and testing sets, and build the MB-DNN: an input layer of D genes feeding a 1000-node shared layer, a 128-node bottleneck, then per-protein 64-node branches with ReLU activations and identity outputs. Optimize an L1 objective with Adam at learning rate 10e-5 for 139 epochs, selecting the optimal model by testing error on the 10% held-out cells, and save the weights as ctpnet_trained_model.h5. For ablation training (e.g., removing SAVER-X), retrain on raw LogNormalize input only, keeping all other hyperparameters fixed so correlation retractions (~0.02 for denoising removal) are attributable to the tweaked parameter.

## Related tools

- **cTP-net** (The multiple-branch deep neural network being trained to map transcriptome to surface protein relative abundance) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoises the scRNA-seq training input before model training; removing it costs ~0.02 correlation)
- **Seurat** (LogNormalize transformation of the denoised counts to produce the network input matrix X)
- **Adam (optimizer)** (Stochastic optimizer of the L1 objective at learning rate 10e-5 for 139 epochs)
- **Python** (Language in which the MB-DNN is constructed and trained via the shared training script) — https://github.com/zhouzilu/cTPnet

## Examples

```
python training_05152020.py  # Adam lr=10e-5, 139 epochs, L1 objective, 90%/10% train/test split -> ctpnet_trained_model.h5
```

## Evaluation signals

- Optimal model is selected by testing error on the 10% held-out cells, not training loss
- Per-protein Pearson correlations between cTP-net-predicted and measured relative abundances on holdout cells are computed and reported
- Ablation check: retraining without SAVER-X denoising should retract correlation by approximately 0.02, larger than any other parameter tweak
- Training runs for 139 epochs with Adam learning rate 10e-5 and an L1 objective, matching the paper's hyperparameters
- Saved weights file ctpnet_trained_model.h5 exists and the network has the expected 1000-node shared layer, 128-node bottleneck, and per-protein 64-node branches

## Limitations

- Removing SAVER-X denoising retracts prediction performance by 0.02 in correlation, more than any other parameter tweak — do not skip denoising for final models
- Comparisons in benchmarks are limited to the same cells as the validation set due to subset variation
- Training on raw protein barcode counts yields worse prediction accuracy than training on relative-abundance-transformed response
- cTP-net is only applicable to UMI-based count expression input; TPM/RPKM accuracy is untested
- No GPU is required, but Windows is not recommended by the maintainers (numpy 1.19.3 needed if attempted)

## Evidence

- [discussion] The objective function was optimized stochastically with Adam46 with learning rate set to 10e–5 for 139 epochs: "The objective function was optimized stochastically with Adam46 with learning rate set to 10e–5 for 139 epochs"
- [discussion] 90/10 train-test split with model selection on testing error: "we keep only 10% of the cells as the testing set, and use the other 90% of the cells for training. The optimal model was selected based on the testing error."
- [discussion] Relative abundance response yields better accuracy than raw counts: "The network trained using this transformed relative protein abundance as the response vector yields better prediction accuracy than the network trained using raw protein barcode counts."
- [discussion] Denoising ablation costs ~0.02 correlation: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [readme] Shared training script availability (README): "we have decided to share our training python code [***here***](https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py)"
