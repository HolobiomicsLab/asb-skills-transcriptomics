---
name: pretrained-checkpoint-shape-validation
description: Use when reconstructing a published deep learning model from its paper's
  architecture description and you possess a pretrained checkpoint (e.g. Zenodo 5895221)
  to validate the reconstruction against layer-by-layer.
when_to_use_negative:
- You only need to run predictions with a pretrained module — pollock predict mode
  loads the module directly and no architectural reconstruction is required
- The checkpoint comes from a different framework (e.g. TensorFlow/Keras) whose state_dict
  semantics do not map onto PyTorch strict loading
- You are training a new module from scratch (pollock train mode) rather than validating
  a published architecture
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: PyTorch
  role: Framework used to implement the encoder, decoder, ZINB/KL/classification loss
    terms, and to strictly load and compare the checkpoint state_dict
- name: Pollock
  role: Source of the architecture definition (VAE with linear classification head),
    CLI hyperparameter defaults, and pretrained module checkpoints
  repo: https://github.com/ding-lab/pollock
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:7578abe93a03cb13c4dd0c63e4dacc5dfddc868e95f89dfecb43ff9a49d53ed7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - Pollock
  - PyTorch
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pretrained-checkpoint-shape-validation

## Summary

Reconstructs Pollock's VAE architecture (fully connected 512→128 layers with ReLU, 64-dim latent embedding, linear classification head with Softmax) in PyTorch and verifies it by strictly loading a pretrained Zenodo checkpoint state_dict with no key or shape mismatches.

## When to use

Use when reconstructing a published deep learning model from its paper's architecture description and you possess a pretrained checkpoint (e.g. Zenodo 5895221) to validate the reconstruction against layer-by-layer.

## When NOT to use

- You only need to run predictions with a pretrained module — pollock predict mode loads the module directly and no architectural reconstruction is required
- The checkpoint comes from a different framework (e.g. TensorFlow/Keras) whose state_dict semantics do not map onto PyTorch strict loading
- You are training a new module from scratch (pollock train mode) rather than validating a published architecture

## Inputs

- pretrained_models.tar.gz from Zenodo record 5895221 (e.g. scRNAseq_generalized module checkpoint)
- Checkpoint state_dict (PyTorch) providing layer names, tensor shapes, and input gene count
- Architecture hyperparameters from Pollock CLI defaults: --middle-dim 512, --encoder-out-dim 128, --latent-dim 64

## Outputs

- architecture_verification_report.txt containing the per-layer dimension comparison between checkpoint and reconstruction
- Strict-load pass/fail result confirming no shape or key mismatches
- Reconstructed PyTorch VAE encoder-decoder with classification head matching the checkpoint

## How to apply

Download pretrained_models.tar.gz from Zenodo record 5895221 and extract a module checkpoint such as scRNAseq_generalized. In PyTorch, implement the encoder as an input layer sized to the gene count, followed by fully connected layers of 512 (--middle-dim default) and 128 (--encoder-out-dim default) with ReLU activations, feeding a 64-dimensional latent embedding (--latent-dim default); mirror it with a decoder trained under a ZINB reconstruction loss (--zinb-scaler), KL regularization (--kl-scaler, default 1e-3), and cross-entropy classification loss (--clf-scaler) summed into one loss. Add the linear classification head (linear + ReLU from the 64-dim latent to Softmax over cell type classes). Read the input gene count from the checkpoint itself, then load the checkpoint's state_dict under strict mode and compare every layer's name and tensor shape. Success is a strict state_dict load with zero mismatches, documented in an architecture_verification_report.txt per-layer dimension comparison.

## Related tools

- **PyTorch** (Framework used to implement the encoder, decoder, ZINB/KL/classification loss terms, and to strictly load and compare the checkpoint state_dict)
- **Pollock** (Source of the architecture definition (VAE with linear classification head), CLI hyperparameter defaults, and pretrained module checkpoints) — https://github.com/ding-lab/pollock

## Examples

```
model.load_state_dict(torch.load('scRNAseq_generalized/checkpoint.pt'), strict=True)
```

## Evaluation signals

- state_dict loads into the reconstructed model in strict mode with no shape or key mismatches
- Per-layer dimension comparison in architecture_verification_report.txt shows matching tensor shapes for every layer name
- Encoder dimensions match CLI defaults: middle layer 512, encoder output 128, latent dimension 64
- Input layer size equals the gene count read from the checkpoint (e.g. gene count consistent with the module's trained gene set)
- Classification head maps the 64-dim latent through linear + ReLU to a Softmax over the module's cell type classes

## Limitations

- Input gene count must be instantiated from the checkpoint; mismatched gene sets (e.g. 4859 genes missing from a dataset after filtering in the paper's example) will prevent a clean strict load
- Verification confirms structural equivalence only — training dynamics or exact weights behavior are not re-validated
- Pollock architecture details rely on CLI defaults; non-default hyperparameters (--middle-dim, --encoder-out-dim, --latent-dim) used at training time must be matched or the load will fail

## Evidence

- [methods] Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head.: "Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head."
- [other] The Pollock network uses two fully connected layers of size 512 and 128, each followed by a ReLU activation function, and a latent cell embedding layer of size 64.: [evidence span withheld — anchor status: not_found]
- [readme] Size of hidden layer in the VAE. Default is 64. --encoder-out-dim ENCODER_OUT_DIM * Size of layer before latent. Default is 128. --middle-dim MIDDLE_DIM * Size of intermediate linear layers. Default is 512.: "Size of hidden layer in the VAE. Default is 64. --encoder-out-dim ENCODER_OUT_DIM * Size of layer before latent. Default is 128. --middle-dim MIDDLE_DIM * Size of intermediate linear layers. Default"
- [methods] The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz: "The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz"
- [readme] This parameter controls how regularized the VAE is. 1e-3 is the default.: "This parameter controls how regularized the VAE is. 1e-3 is the default."
