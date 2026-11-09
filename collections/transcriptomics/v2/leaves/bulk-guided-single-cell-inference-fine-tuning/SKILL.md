---
name: bulk-guided-single-cell-inference-fine-tuning
description: Use when a cohort has bulk RNA-seq data for all samples plus processed
  single-cell data from one representative sample, and single-cell profiles must be
  inferred for the remaining target samples.
when_to_use_negative:
- No single-cell reference from a representative is available — the tool's deep generative
  model enables 'single-cell level deconvolution' provided there is a single-cell
  reference
- Goal is only cell-type proportion estimation from bulk data — conventional deconvolution
  tools (CIBERSORTx, Bisque, DWLS, MuSiC, NNLS) suffice at cell-type-level resolution
- Input is another modality (e.g., proteomics) — extension to other biological modalities
  is only future work, not yet implemented
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSemiProfiler
  role: Implements the VAE-GAN pretraining, bulk-guided fine-tuning, and in silico
    inference pipeline that produces target-sample single-cell profiles
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: PyTorch
  role: Deep learning backend used to train the VAE-GAN generator/discriminator during
    pretraining and inference fine-tuning
- name: Scikit-learn
  role: Provides PCA used for dimensionality reduction in the representative processing
    stage feeding the model
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:e2f39ae3bbf0e6c342951c04f1cebef952b1119d587bdd7f62cc359b27702276
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - PyTorch
  - Scikit-learn
  - scSemiProfiler
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Bulk-guided single-cell inference fine-tuning

## Summary

Fine-tunes a pretrained VAE-GAN generator so that generated single-cell profiles match a target sample's real bulk sequencing data, enabling in silico inference of single-cell data for cohort samples that were never directly sequenced.

## When to use

Use when a cohort has bulk RNA-seq data for all samples plus processed single-cell data from one representative sample, and single-cell profiles must be inferred for the remaining target samples.

## When NOT to use

- No single-cell reference from a representative is available — the tool's deep generative model enables 'single-cell level deconvolution' provided there is a single-cell reference
- Goal is only cell-type proportion estimation from bulk data — conventional deconvolution tools (CIBERSORTx, Bisque, DWLS, MuSiC, NNLS) suffice at cell-type-level resolution
- Input is another modality (e.g., proteomics) — extension to other biological modalities is only future work, not yet implemented

## Inputs

- Preprocessed bulk RNA-seq count matrix for the cohort (e.g., COVID-19 dataset, preprocessed as detailed by ref 45)
- Processed representative single-cell data (e.g., AP6) augmented with MSigDB gene set scores (.h5ad)
- Bulk sequencing data for the target sample (e.g., AP1)

## Outputs

- Inferred single-cell profiles for the target sample (inferred_cells.h5ad)
- Trained generator checkpoint file

## How to apply

First pretrain the VAE-GAN on the representative's single-cell data (e.g., AP6 cells with MSigDB gene set scores, ZINB decoder, latent dim 32, 256-neuron hidden layers, lr 1e-3, K=15 neighbor aggregation for the GCN layer), using a two-stage pretraining that adds the representative pseudobulk loss LBulkR at lr 1e-4. For fine-tuning, remove the discriminator and minimize LInference = LVAE + λBulkT·LBulkT, where the target pseudobulk estimate uses conversion ratio α = (BT+ε)⊘(BR+ε) with pseudocount ε = 0.1. Run gradient-threshold mini-stages at max_expr, 1/2, 1/4, 1/8, and 1/(8√2) of the representative's peak expression, quadrupling the bulk-loss weight at each stage (lr 2e-4, ≤150 epochs per mini-stage). Apply the trained generator to produce the inferred profiles for the target sample (e.g., AP1) and save them to inferred_cells.h5ad together with the generator checkpoint.

## Related tools

- **scSemiProfiler** (Implements the VAE-GAN pretraining, bulk-guided fine-tuning, and in silico inference pipeline that produces target-sample single-cell profiles) — https://github.com/mcgilldinglab/scSemiProfiler
- **PyTorch** (Deep learning backend used to train the VAE-GAN generator/discriminator during pretraining and inference fine-tuning)
- **Scikit-learn** (Provides PCA used for dimensionality reduction in the representative processing stage feeding the model)

## Examples

```
conda create -n semiprofiler python=3.9; conda activate semiprofiler; pip install --upgrade https://github.com/mcgilldinglab/scSemiProfiler/zipball/main
```

## Evaluation signals

- Pearson correlation between inferred and true target-sample cells exceeds the representative-vs-target baseline (0.904 vs 0.568 reported for AP1 vs AP6)
- Script runs end-to-end over the training/inference path and emits both inferred_cells.h5ad and the generator checkpoint
- Fine-tuning loss decreases across the gradient-threshold mini-stages (max_expr → 1/(8√2) of peak expression) as λBulkT is quadrupled
- Reconstructed pseudobulk of inferred cells matches the α-converted target pseudobulk (L2-norm LBulkT minimized)

## Limitations

- The tool requires a single-cell reference to perform single-cell level deconvolution
- Extension to proteomics and other modalities is only future work, not yet implemented; algorithm adjustments and benchmarking against established standards would be required
- No software versions stated for scSemiProfiler or dependencies in the discussion section

## Evidence

- [readme] Fine-tuning incorporates target bulk: [evidence span withheld — anchor status: external_doc]
- [discussion] VAE-GAN infers cohort remainder: "employs a deep generative learning model, VAE-GAN, to infer the single-cell data of the remainder of the cohort."
- [methods] In silico inference fine-tuning step: "Fine-Tune The Deep Generative Learning Model To Infer The Single-Cell Measurements For The Target Samples Blue (In Silico Inference)"
- [discussion] Single-cell reference prerequisite: "The tool’s deep generative model enables “single-cell level deconvolution” across various studies, provided there is a single-cell reference."
- [methods] Gene set score augmentation of reference cells: "gene set scores are calculated using all curated gene sets from MSigDB59 and Ernst et al.91"
