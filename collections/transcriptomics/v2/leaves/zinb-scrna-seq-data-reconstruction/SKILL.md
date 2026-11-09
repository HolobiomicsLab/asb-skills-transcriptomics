---
name: zinb-scrna-seq-data-reconstruction
description: Use when you have bulk RNA-seq data for cohort samples plus a single-cell
  reference (e.g., sequenced representative AP6) and need inferred single-cell profiles
  (.h5ad) for target samples at true single-cell resolution.
when_to_use_negative:
- No single-cell reference is available — the tool requires a single-cell reference
  for single-cell level deconvolution
- Goal is bulk deconvolution only to cell-type proportions — classical methods (CIBERSORTx,
  Bisque, DWLS, MuSiC, NNLS) suffice and this deep generative route is unnecessary
- Data modalities beyond scRNA-seq (e.g., proteomics) — extension is future work and
  algorithms would require adjustment
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scSemiProfiler
  role: Framework implementing the three-stage VAE-GAN pretraining/fine-tuning workflow
    for in silico single-cell inference
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: PyTorch
  role: Deep learning backend used to train the VAE-GAN generator/discriminator with
    the ZINB decoder
- name: Scikit-learn
  role: Used in the surrounding pipeline for PCA dimensionality reduction of bulk
    features feeding the model
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:1f2aa51a97f53946da4ba4a668a2bd1a391b582e6af1d0c9d45d69902dda6101
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - PyTorch
  - Scikit-learn
  - scSemiProfiler
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# zinb-scRNA-seq-data-reconstruction

## Summary

A three-stage VAE-GAN deep generative workflow (representative pretraining with a ZINB decoder, pseudobulk-anchored pretraining, and target-bulk fine-tuning) that reconstructs single-cell profiles for cohort samples that were never directly sequenced, as implemented in scSemiProfiler.

## When to use

Use when you have bulk RNA-seq data for cohort samples plus a single-cell reference (e.g., sequenced representative AP6) and need inferred single-cell profiles (.h5ad) for target samples at true single-cell resolution.

## When NOT to use

- No single-cell reference is available — the tool requires a single-cell reference for single-cell level deconvolution
- Goal is bulk deconvolution only to cell-type proportions — classical methods (CIBERSORTx, Bisque, DWLS, MuSiC, NNLS) suffice and this deep generative route is unnecessary
- Data modalities beyond scRNA-seq (e.g., proteomics) — extension is future work and algorithms would require adjustment

## Inputs

- Preprocessed COVID-19 bulk count matrix for the cohort (including target sample AP1)
- Processed representative (AP6) single-cell data with MSigDB gene set scores (.h5ad)
- K-nearest-neighbor graph parameters (K=15) for the GCN layer

## Outputs

- inferred_cells.h5ad containing inferred single-cell profiles for the target sample
- Trained generator checkpoint file
- Reconstructed representative single-cell profiles from pretraining stages

## How to apply

Stage 1 pretrains the VAE-GAN generator on the representative's processed cells (e.g., COVID-19 representative AP6 augmented with MSigDB gene set scores, K=15 nearest-neighbor aggregation for the GCN layer) using a ZINB decoder, latent dimension 32, 256-neuron hidden layers, and SCVI default learning rate 1e-3, training the generator alone ~100 epochs then alternating generator/discriminator every 3 epochs for 100 iterations. Stage 2 adds a representative pseudobulk loss LBulkR (L2 norm between reconstructed and real pseudobulk) with full-batch training at learning rate 1e-4, alternating every 3 epochs for 50 iterations. Stage 3 removes the discriminator and minimizes LInference = LVAE + λBulkT·LBulkT, where the target pseudobulk estimate uses conversion ratio α = (BT+ε)⊘(BR+ε) with pseudocount ε = 0.1; run gradient-threshold mini-stages at max_expr, 1/2, 1/4, 1/8, and 1/(8√2) of the representative's peak expression, quadrupling the bulk-loss weight at each stage, learning rate 2e-4, at most 150 epochs per mini-stage. The trained generator is then applied to produce inferred cells for the target (e.g., AP1) and saved to inferred_cells.h5ad together with the generator checkpoint.

## Related tools

- **scSemiProfiler** (Framework implementing the three-stage VAE-GAN pretraining/fine-tuning workflow for in silico single-cell inference) — https://github.com/mcgilldinglab/scSemiProfiler
- **PyTorch** (Deep learning backend used to train the VAE-GAN generator/discriminator with the ZINB decoder)
- **Scikit-learn** (Used in the surrounding pipeline for PCA dimensionality reduction of bulk features feeding the model)

## Examples

```
conda activate semiprofiler; pip install --upgrade https://github.com/mcgilldinglab/scSemiProfiler/zipball/main
```

## Evaluation signals

- Pearson correlation between inferred and true target cells exceeds the representative-vs-target baseline (0.904 vs 0.568 reported for AP1)
- Script runs end-to-end over the pretraining (stages 1–2) and inference (stage 3) path without error, emitting inferred_cells.h5ad plus the generator checkpoint
- Stage-1 generator converges under the ZINB reconstruction loss before discriminator alternation (every 3 epochs, 100 iterations) begins
- Semi-profiled data resembles real-profiled ground truth in UMAP visualization and cell type proportions

## Limitations

- Requires a single-cell reference sample (real or public) to guide the generative model
- Only valid for scRNA-seq; extension to proteomics and other modalities is future work requiring algorithm adjustments and benchmarking
- No software versions for scSemiProfiler or dependencies stated in the article discussion section

## Evidence

- [methods] Pretrain The Deep Generative Learning Model For Reconstructing The Single-Cell Data Of The Selected Representatives (Representative Reconstruction): "Pretrain The Deep Generative Learning Model For Reconstructing The Single-Cell Data Of The Selected Representatives (Representative Reconstruction)"
- [methods] Fine-Tune The Deep Generative Learning Model To Infer The Single-Cell Measurements For The Target Samples: "Fine-Tune The Deep Generative Learning Model To Infer The Single-Cell Measurements For The Target Samples Blue (In Silico Inference)"
- [readme] a deep generative model first learns the distribution of the reference single-cell data, generating reconstructions of the reference cells: [evidence span withheld — anchor status: external_doc]
- [discussion] employs a deep generative learning model, VAE-GAN, to infer the single-cell data of the remainder of the cohort: "employs a deep generative learning model, VAE-GAN, to infer the single-cell data of the remainder of the cohort."
