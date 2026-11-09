---
name: vae-gan-single-cell-generative-modeling
description: Use when you have bulk RNA-seq for a cohort plus single-cell data for
  one representative sample and need single-cell-resolution profiles (e.g., .h5ad)
  for non-sequenced target samples.
when_to_use_negative:
- You need only cell-type proportions from bulk data — conventional deconvolution
  methods (CIBERSORTx, Bisque, DWLS, MuSiC, NNLS) suffice and cannot reach true single-cell
  resolution, whereas this skill targets single-cell profile generation.
- No single-cell reference exists for any similar sample — the model requires a single-cell
  reference to perform single-cell level deconvolution.
- You want to extend to proteomics or other modalities — this is future work and would
  require algorithm adjustments and new benchmarking.
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scSemiProfiler
  role: Framework implementing the VAE-GAN training (representative reconstruction
    and in silico inference fine-tuning) that generates target single-cell profiles
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: PyTorch
  role: Deep learning backend for the VAE-GAN generator/discriminator training
  repo: https://pytorch.org/get-started/locally/
- name: Scikit-learn
  role: Supplies PCA and K-Means used to prepare features and clusters feeding the
    model
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:d621d328769acabd285549dcf0de1ba98a26b18f67e8f5b27dcca92a3461c335
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# vae-gan-single-cell-generative-modeling

## Summary

A VAE-GAN deep generative approach (as implemented in scSemiProfiler) that learns the distribution of a representative single-cell reference (with ZINB decoding) and then fine-tunes the generator against a target sample's bulk RNA-seq to infer single-cell profiles for cohort samples never directly sequenced.

## When to use

Use when you have bulk RNA-seq for a cohort plus single-cell data for one representative sample and need single-cell-resolution profiles (e.g., .h5ad) for non-sequenced target samples.

## When NOT to use

- You need only cell-type proportions from bulk data — conventional deconvolution methods (CIBERSORTx, Bisque, DWLS, MuSiC, NNLS) suffice and cannot reach true single-cell resolution, whereas this skill targets single-cell profile generation.
- No single-cell reference exists for any similar sample — the model requires a single-cell reference to perform single-cell level deconvolution.
- You want to extend to proteomics or other modalities — this is future work and would require algorithm adjustments and new benchmarking.

## Inputs

- Preprocessed bulk RNA-seq count matrix for the cohort (e.g., COVID-19 bulk counts)
- Processed representative single-cell data (AnnData/.h5ad) augmented with MSigDB gene set scores
- Bulk sequencing data for the target sample and for the representative sample (to estimate conversion ratio α)

## Outputs

- inferred_cells.h5ad containing inferred single-cell profiles for the target sample
- Saved generator checkpoint file from the fine-tuned VAE-GAN

## How to apply

Pretrain stage 1 trains the VAE-GAN generator to reconstruct representative cells (e.g., AP6) with a ZINB decoder, latent dimension 32, 256-neuron hidden layers, SCVI default learning rate 1e-3, generator alone for ~100 epochs then alternated with the discriminator every 3 epochs for 100 iterations; MSigDB gene set scores and K=15 nearest-neighbor aggregation feed the GCN layer. Pretrain stage 2 adds a representative pseudobulk loss (L2 norm between reconstructed and real pseudobulk) with learning rate reduced to 1e-4, alternating for 50 iterations. For in silico inference of a target (e.g., AP1), remove the discriminator and minimize LInference = LVAE + λBulkT·LBulkT, where the target pseudobulk estimate uses conversion ratio α = (BT+ε)⊘(BR+ε) with pseudocount ε = 0.1; run gradient-threshold mini-stages at max_expr, 1/2, 1/4, 1/8, and 1/(8√2) of the representative's peak expression, quadrupling the bulk-loss weight each stage, with learning rate 2e-4 and at most 150 epochs per mini-stage. Rationale: tying the generator to the target's real bulk data via LBulkT enables precise in silico inference rather than mere reconstruction of the reference.

## Related tools

- **scSemiProfiler** (Framework implementing the VAE-GAN training (representative reconstruction and in silico inference fine-tuning) that generates target single-cell profiles) — https://github.com/mcgilldinglab/scSemiProfiler
- **PyTorch** (Deep learning backend for the VAE-GAN generator/discriminator training) — https://pytorch.org/get-started/locally/
- **Scikit-learn** (Supplies PCA and K-Means used to prepare features and clusters feeding the model)

## Examples

```
conda create -n semiprofiler python=3.9; conda activate semiprofiler; pip install --upgrade https://github.com/mcgilldinglab/scSemiProfiler/zipball/main
```

## Evaluation signals

- Pearson correlation between inferred and true target cells exceeds the representative-vs-target baseline (0.904 vs 0.568 reported for COVID-19 AP1 vs AP6).
- Script runs end-to-end over the training/inference path (pretrain stage 1 → stage 2 → fine-tune → save).
- Bulk loss LBulkT decreases across gradient-threshold mini-stages, confirming the generated pseudobulk converges toward the target bulk profile.
- Output inferred_cells.h5ad loads as valid AnnData and its UMAP/cell-type proportions resemble the real-profiled ground truth.
- Stage transitions follow the specified schedule: LR 1e-3 → 1e-4 → 2e-4 and discriminator removed before fine-tuning.

## Limitations

- The tool requires a single-cell reference to perform single-cell level deconvolution.
- Extension to proteomics and other modalities is only future work; algorithm adjustments and comprehensive benchmarking against established standards are still required.
- No software versions stated for scSemiProfiler or its dependencies; cost savings depend on sequencing representative samples (e.g., 1/10 to 1/3 cost, 66.3% savings in the COVID-19 example).

## Evidence

- [other] VAE-GAN model is fine-tuned with a target bulk loss tied to the target sample's real bulk sequencing data, which enables precise in silico inference: [evidence span withheld — anchor status: not_found]
- [discussion] employs a deep generative learning model, VAE-GAN, to infer the single-cell data of the remainder of the cohort: "employs a deep generative learning model, VAE-GAN, to infer the single-cell data of the remainder of the cohort."
- [readme] For each target sample, a deep generative model first learns the distribution of the reference single-cell data: [evidence span withheld — anchor status: external_doc]
- [methods] gene set scores are calculated using all curated gene sets from MSigDB59: "gene set scores are calculated using all curated gene sets from MSigDB59 and Ernst et al.91"
