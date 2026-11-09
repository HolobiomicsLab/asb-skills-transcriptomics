---
name: map-inference-of-count-generative-models
description: Use when fitting count-based generative topic models (e.g., MIRA RP models)
  to single-cell multiome data and you need to compare model fit — via held-out log-likelihood
  — across single-batch vs multi-batch training, with or without technical effect
  correction.
when_to_use_negative:
- Data are from a single batch with no batch-confounded cell states — per-batch and
  pooled regimes collapse to the same condition and CODAL augmentation adds no correction
  signal.
- The generative model is not count-based (e.g., already reduced to normalized/log-transformed
  feature tables such as normalized_total=10000 log+1 values) — the Negative Binomial
  likelihood and fixed technical effect vectors require raw counts.
- The analysis goal is batch correction of latent embeddings rather than likelihood-based
  comparison of generative model fit — other benchmarked methods (scVI, Harmony, FastMNN,
  Scanorama, Seurat v3) address that use case.
edam_operation: http://edamontology.org/operation_0496
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MIRA
  role: Provides the RP topic model architecture, MAP parameter inference procedure,
    and held-out log-likelihood evaluation for the count-based generative models
  repo: https://github.com/cistrome/MIRA
- name: CODAL
  role: Supplies fixed technical effect vectors t (non-trainable additive component
    of the Negative Binomial distribution), the kappa_i softmax denominator approximation,
    and unconfounded compositional accessibility rates used as model input
  repo: https://github.com/cistrome/MIRA
- name: scanpy
  role: 'Preprocessing of the multimodal dataset: count filtering (cells < 400 counts,
    genes < 30 counts), normalize_total with target_sum=10000, log+1 transformation,
    HVG selection'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:4181b89e44926fdcaa2c078ea8c49388a110629084ad6261fb141a763da8b6e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cistrome/MIRA
  tools:
  - CODAL
  - MIRA
  - scanpy
  edam_operation: http://edamontology.org/operation_0496
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# MAP inference of count generative models

## Summary

Fit MIRA regulatory potential (RP) count-based generative models via MAP parameter inference, optionally fixing CODAL-estimated technical effect vectors as a non-trainable additive component of the Negative Binomial generative distribution, and compare held-out log-likelihood across training regimes.

## When to use

Use when fitting count-based generative topic models (e.g., MIRA RP models) to single-cell multiome data and you need to compare model fit — via held-out log-likelihood — across single-batch vs multi-batch training, with or without technical effect correction.

## When NOT to use

- Data are from a single batch with no batch-confounded cell states — per-batch and pooled regimes collapse to the same condition and CODAL augmentation adds no correction signal.
- The generative model is not count-based (e.g., already reduced to normalized/log-transformed feature tables such as normalized_total=10000 log+1 values) — the Negative Binomial likelihood and fixed technical effect vectors require raw counts.
- The analysis goal is batch correction of latent embeddings rather than likelihood-based comparison of generative model fit — other benchmarked methods (scVI, Harmony, FastMNN, Scanorama, Seurat v3) address that use case.

## Inputs

- NEURIPS 10x Multiome bone marrow single-cell multimodal count matrices (scRNA-seq + scATAC-seq), batch label per cell
- Gene set of 2841 highly variable genes with UCSC TSS annotations
- Pre-fitted CODAL technical effect vectors t and compositional accessibility rates
- scanpy-preprocessed AnnData objects with cell/gene count filters applied (cells < 400 counts and genes < 30 counts removed)

## Outputs

- Trained MIRA RP models for each training regime (13 per-batch, pooled-uncorrected, pooled-CODAL-augmented)
- Held-out log-likelihood scores per batch and regime
- Batch-by-regime log-likelihood matrix rendered as a heatmap (Fig. 4b,c comparison)

## How to apply

Preprocess the multimodal NEURIPS 10x Multiome bone marrow data with scanpy and restrict the RP model to the 2841 genes that are both highly variable and have UCSC TSS annotations. Partition each of the 13 sequencing batches into 4:1 training/test splits, then train RP models by MAP inference: (a) 13 per-batch models without augmentation, (b) one pooled model across all batches without augmentation, and (c) one pooled model with CODAL augmentation in which the CODAL-estimated technical effect vectors t are fixed as a non-trainable additive component of the Negative Binomial generative distribution, with the softmax denominator kappa_i approximated from the CODAL model and accessibility inputs taken from CODAL's unconfounded compositional rates. Evaluate the log-likelihood of every trained model on the held-out test cells of each of the 13 batches and assemble scores into a batch-by-regime matrix rendered as a heatmap (reproducing the Fig. 4b,c comparison). The rationale is that explicitly disentangling technical from biological effects via mutual information regularization lets a single generative model generalize across batches better than per-batch or uncorrected pooled fits.

## Related tools

- **MIRA** (Provides the RP topic model architecture, MAP parameter inference procedure, and held-out log-likelihood evaluation for the count-based generative models) — https://github.com/cistrome/MIRA
- **CODAL** (Supplies fixed technical effect vectors t (non-trainable additive component of the Negative Binomial distribution), the kappa_i softmax denominator approximation, and unconfounded compositional accessibility rates used as model input) — https://github.com/cistrome/MIRA
- **scanpy** (Preprocessing of the multimodal dataset: count filtering (cells < 400 counts, genes < 30 counts), normalize_total with target_sum=10000, log+1 transformation, HVG selection)

## Examples

```
conda create --name mira-env -c conda-forge -c pytorch -c bioconda scanpy jupyter leidenalg && conda activate mira-env && pip3 install mira-multiome
```

## Evaluation signals

- The 'all batches + CODAL technical effect correction' regime attains the highest total held-out log-likelihood across the 13 batches, matching the paper's Fig. 4b,c regulatory potential result.
- The augmented pooled model also outperforms per-batch and uncorrected all-batch models broadly across all genes, not just in aggregate.
- The gene set used for RP model fitting contains exactly the 2841 genes that are both highly variable and UCSC TSS-annotated.
- The batch-by-regime matrix covers all 13 sequencing batches × 3 training regimes (per-batch, pooled-uncorrected, pooled-CODAL-augmented) with 4:1 train/test splits per batch.
- CODAL technical effect vectors are non-trainable (fixed) during MAP inference of the augmented model; only biological/other parameters update.

## Limitations

- The kappa_i softmax denominator is approximated from the CODAL model rather than learned, an approximation of the true generative likelihood.
- The MAP-inferred augmented model assumes variance in technical effects is driven primarily by batch-specific factors, using batch of origin only as a proxy variable.
- Authors caution that batch-confounded or perturbation-induced states inferred by such models should be validated with orthogonal biological evidence to rule out under-correction of technical effects, and CODAL does not replace good experimental design with appropriate controls.

## Evidence

- [other] RP models with CODAL technical effect augmentation on the pooled training cells by fixing the CODAL-estimated technical effect vectors t as a non-trainable additive component of the Negative Binomial generative distribution: [evidence span withheld — anchor status: not_found]
- [methods] MAP inference restriction to 2841 TSS-annotated HVG genes: "2841 genes which were both highly variable and had UCSC TSS annotations78"
- [abstract] CODAL enables generalization of count-based generative models to multi-batch data: "and enables the generalization of other count-based generative models to multi-batched data"
- [readme] MIRA topic models all use CODAL algorithm for inference: [evidence span withheld — anchor status: external_doc]
- [intro] Batch-of-origin proxy assumption for technical effects: "the cells’ batch of origin can be used as a proxy variable to indicate which cells were subjected to a protocol using common reagents and under similar conditions"
