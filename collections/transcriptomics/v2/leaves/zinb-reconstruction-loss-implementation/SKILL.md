---
name: zinb-reconstruction-loss-implementation
description: 'Use when reconstructing, retraining, or verifying Pollock''s VAE in
  PyTorch: the decoder must reproduce counts via a ZINB loss weighted by --zinb-scaler,
  alongside KL and classification terms.'
when_to_use_negative:
- Input expression matrix is normalized/log-transformed (e.g. after sc.pp.normalize_total
  + log1p) — the ZINB reconstruction loss expects raw counts; the README states raw
  counts must be stored in the input object
- The goal is prediction or explanation with an existing pretrained module rather
  than reconstructing or retraining the VAE loss
- Data modality is snATAC-seq gene activity where count-based ZINB assumptions differ
  from the scRNA-seq setting the defaults were tuned on
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: Pollock
  role: Framework whose VAE encoder/decoder with ZINB reconstruction, KL, and classification
    losses is being implemented; pretrained modules downloaded from Zenodo
  repo: github.com/ding-lab/pollock
- name: PyTorch
  role: Framework used to implement the encoder, ZINB decoder, classification head,
    and combined loss
- name: scanpy
  role: Loads scRNA-seq input (.h5ad / 10x mtx) whose raw counts feed the ZINB loss
    during training
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:071a5b27872a508698df2576354abaed5e908d67c71741c75b09f5c7832f88e7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pollock
  - PyTorch
  - scanpy
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# zinb-reconstruction-loss-implementation

## Summary

Implement the zero-inflated negative binomial (ZINB) reconstruction loss of the Pollock VAE, weighted by the --zinb-scaler argument and summed with the KL regularization (default 1e-3) and classification cross-entropy losses into a single overall loss for scRNA-seq cell type classification.

## When to use

Use when reconstructing, retraining, or verifying Pollock's VAE in PyTorch: the decoder must reproduce counts via a ZINB loss weighted by --zinb-scaler, alongside KL and classification terms.

## When NOT to use

- Input expression matrix is normalized/log-transformed (e.g. after sc.pp.normalize_total + log1p) — the ZINB reconstruction loss expects raw counts; the README states raw counts must be stored in the input object
- The goal is prediction or explanation with an existing pretrained module rather than reconstructing or retraining the VAE loss
- Data modality is snATAC-seq gene activity where count-based ZINB assumptions differ from the scRNA-seq setting the defaults were tuned on

## Inputs

- Raw (unnormalized) single-cell count matrix — Seurat .rds (@assays$RNA@counts), scanpy .h5ad (.X must be raw counts), or 10x cellranger output
- Training hyperparameters: --zinb-scaler, --kl-scaler (default 1e-3), --clf-scaler, --latent-dim 64, --encoder-out-dim 128, --middle-dim 512, --epochs 20
- Pretrained module checkpoint (e.g. scRNAseq_generalized from pretrained_models.tar.gz, Zenodo record 5895221)

## Outputs

- Trained Pollock VAE module with ZINB decoder, 64-dim latent embeddings, and cell type Softmax probabilities
- architecture_verification_report.txt containing per-layer dimension comparison against the checkpoint state_dict

## How to apply

In PyTorch, implement the encoder as an input layer sized to the gene count, fully connected layers of size 512 (--middle-dim default) and 128 (--encoder-out-dim default) each followed by ReLU, feeding a 64-dimensional latent cell embedding (--latent-dim default). Implement the mirrored decoder whose reconstruction output is scored with the ZINB loss, weighted by --zinb-scaler; add the KL regularization term weighted by --kl-scaler (default 1e-3) and the classification cross-entropy loss from the linear + ReLU head (64-dim latent to Softmax over cell type classes) weighted by --clf-scaler, summed into a single overall loss. The input expression matrix must be raw counts (unnormalized) since the ZINB loss models count data. To verify, load a pretrained checkpoint (e.g. scRNAseq_generalized from Zenodo record 5895221) with a strict state_dict load and confirm no shape or key mismatches in the architecture_verification_report.txt.

## Related tools

- **Pollock** (Framework whose VAE encoder/decoder with ZINB reconstruction, KL, and classification losses is being implemented; pretrained modules downloaded from Zenodo) — github.com/ding-lab/pollock
- **PyTorch** (Framework used to implement the encoder, ZINB decoder, classification head, and combined loss)
- **scanpy** (Loads scRNA-seq input (.h5ad / 10x mtx) whose raw counts feed the ZINB loss during training)

## Examples

```
pollock train from_scanpy --module-filepath out_module --scanpy-h5ad-filepath counts.h5ad --cell-type-key cell_type --epochs 20 --latent-dim 64 --encoder-out-dim 128 --middle-dim 512 --kl-scaler 1e-3
```

## Evaluation signals

- Strict state_dict load of a pretrained checkpoint (e.g. scRNAseq_generalized) into the reconstructed model completes with no shape or key mismatches
- architecture_verification_report.txt shows every checkpoint layer name and tensor shape matches the reconstruction, with input dimension equal to the gene count read from the checkpoint
- Latent embedding dimension is 64, encoder-out 128, middle layers 512 — matching --latent-dim, --encoder-out-dim, --middle-dim defaults
- Overall loss equals the weighted sum: --zinb-scaler * ZINB reconstruction + --kl-scaler * KL (default 1e-3) + --clf-scaler * classification cross-entropy
- Training with --epochs 20 (default) and --n-per-cell-type 500 validation split converges and yields cell type predictions consistent with the module's trained cell type classes

## Limitations

- The ZINB reconstruction term requires raw counts; normalized inputs (e.g. normalize_total to 1e4 + log1p) violate its assumptions
- Increasing --kl-scaler makes cell embeddings noisier but typically more generalizable; decreasing it reduces noise but reduces generalizability
- Default architecture assumes fixed layer sizes (512/128/64) — checkpoints trained with non-default hyperparameters require matching dimensions when instantiating the model
- Explain-mode style operations have long runtimes for large cell numbers (~15 minutes for ~100 cells with ~100 background samples), though this affects the downstream head rather than the loss itself

## Evidence

- [methods] Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head.: "Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head."
- [methods] The Pollock network uses two fully connected layers of size 512 and 128, each followed by a ReLU activation function, and a latent cell embedding layer of size 64.: [evidence span withheld — anchor status: not_found]
- [other] Implement the mirrored decoder with a ZINB reconstruction loss weighted by --zinb-scaler, the KL regularization term weighted by --kl-scaler (default 1e-3): [evidence span withheld — anchor status: not_found]
- [readme] This parameter controls how regularized the VAE is. 1e-3 is the default. If increased the cell embeddings are typically more noisy, but typically moregeneralizable.: "This parameter controls how regularized the VAE is. 1e-3 is the default. If increased the cell embeddings are typically more noisy, but typically moregeneralizable"
- [other] Load the pretrained checkpoint's state_dict into the implemented module and compare every layer's name and tensor shape against the reconstruction: [evidence span withheld — anchor status: not_found]
- [readme] Raw RNA-seq (i.e. not normalized) counts must be stored in @assays$RNA@counts or @assays$RNA@counts.: "Raw RNA-seq (i.e. not normalized) counts must be stored in @assays$RNA@counts or @assays$RNA@counts."
