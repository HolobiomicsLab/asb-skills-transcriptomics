---
name: technical-effect-augmentation-of-generative-models
description: Use when a count-based generative model (Negative Binomial likelihood,
  e.g., MIRA RP topic models) must be trained or evaluated on multi-batch single-cell
  data where technical effects would otherwise distort the held-out likelihood comparison.
when_to_use_negative:
- Data contains a single batch with no technical effect to disentangle — per-batch
  training suffices and CODAL augmentation adds no benefit
- The generative model is not count-based (e.g., Seurat v3, FastMNN, Harmony, Scanorama
  integration methods), since the fixed technical vectors are defined as an additive
  component of the Negative Binomial distribution
- Cells are batch-confounded but no control group unaffected by the perturbation exists
  in both treatment and control batches, making technical effect estimation unreliable
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3295
tools:
- name: MIRA
  role: Provides the regulatory potential topic model whose Negative Binomial generative
    distribution is augmented with fixed CODAL technical effect vectors, trained via
    MAP inference
  repo: https://github.com/cistrome/MIRA
- name: CODAL
  role: Estimates the technical effect vectors t and unconfounded compositional accessibility
    rates that are fixed into the MIRA generative distribution
  repo: https://github.com/cistrome/MIRA
- name: scanpy
  role: 'Preprocessing of the NEURIPS multimodal dataset: normalize_total (target_sum=10000),
    log+1 transform, highly variable gene selection (min dispersion 0.3)'
  repo: https://scanpy.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:e6a4df1b25fb39df9893bffa51d28f670f8d34015f5b07ab188dc5587b2b0d3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cistrome/MIRA
  tools:
  - CODAL
  - MIRA
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# technical-effect-augmentation-of-generative-models

## Summary

Augments count-based single-cell generative models (e.g., MIRA regulatory potential topic models) by fixing CODAL-estimated technical effect vectors as non-trainable additive components of the Negative Binomial generative distribution, enabling valid multi-batch inference. This generalizes count-based models like MIRA to batched single-cell data without confounding biological signal.

## When to use

Use when a count-based generative model (Negative Binomial likelihood, e.g., MIRA RP topic models) must be trained or evaluated on multi-batch single-cell data where technical effects would otherwise distort the held-out likelihood comparison.

## When NOT to use

- Data contains a single batch with no technical effect to disentangle — per-batch training suffices and CODAL augmentation adds no benefit
- The generative model is not count-based (e.g., Seurat v3, FastMNN, Harmony, Scanorama integration methods), since the fixed technical vectors are defined as an additive component of the Negative Binomial distribution
- Cells are batch-confounded but no control group unaffected by the perturbation exists in both treatment and control batches, making technical effect estimation unreliable

## Inputs

- NEURIPS 10x Multiome bone marrow multimodal count matrices (scRNA-seq + scATAC-seq, 13 sequencing batches) in AnnData format
- Highly variable genes with UCSC TSS annotations (2841 genes, min dispersion 0.3)
- Pre-trained CODAL model supplying fixed technical effect vectors t and compositional accessibility rates
- Train/test cell partition per batch (4:1 ratio)

## Outputs

- Held-out log-likelihood scores per trained model on test cells of each of the 13 batches
- Batch-by-regime log-likelihood matrix rendered as a heatmap (reproducing Fig. 4b,c)
- Trained regulatory potential topic models per regime (per-batch, pooled-uncorrected, pooled-augmented)

## How to apply

First run CODAL on the batched dataset to estimate batch-specific technical effect vectors t. Then train the MIRA regulatory potential model restricted to the 2841 genes that are both highly variable and have UCSC TSS annotations, using MAP parameter inference on pooled training cells (4:1 train/test split per batch). Fix the CODAL-estimated technical effect vectors t as a non-trainable additive component of the Negative Binomial generative distribution, approximating the softmax denominator kappa_i from the CODAL model, and feed CODAL's unconfounded compositional accessibility rates as the accessibility input. Train three regimes for comparison: per-batch models (13 single-batch models), all-batches uncorrected, and all-batches with CODAL technical effect augmentation; evaluate each model's log-likelihood on held-out test cells of each of the 13 NEURIPS bone marrow sequencing batches and assemble a batch-by-regime likelihood heatmap.

## Related tools

- **MIRA** (Provides the regulatory potential topic model whose Negative Binomial generative distribution is augmented with fixed CODAL technical effect vectors, trained via MAP inference) — https://github.com/cistrome/MIRA
- **CODAL** (Estimates the technical effect vectors t and unconfounded compositional accessibility rates that are fixed into the MIRA generative distribution) — https://github.com/cistrome/MIRA
- **scanpy** (Preprocessing of the NEURIPS multimodal dataset: normalize_total (target_sum=10000), log+1 transform, highly variable gene selection (min dispersion 0.3)) — https://scanpy.readthedocs.io/en/stable/

## Examples

```
conda create --name mira-env -c conda-forge -c pytorch -c bioconda scanpy jupyter leidenalg && pip3 install mira-multiome
```

## Evaluation signals

- The 'all batches + CODAL technical effect correction' regime must attain the highest total held-out log-likelihood across all 13 batches, matching the paper's Fig. 4b,c result
- The augmented model must also show broadly highest held-out likelihood across all genes, not just per-batch totals
- The 2841-gene restriction must hold (highly variable genes intersected with UCSC TSS annotations) before training the RP model
- Technical effect vectors must remain non-trainable (fixed) during MIRA training; only the biological topic parameters update via MAP inference
- Train/test splits must preserve a 4:1 ratio within each of the 13 batches for evaluation consistency

## Limitations

- Authors caution that new batch-confounded or perturbation-induced cell states should be validated with orthogonal biological evidence to rule out under-correction of technical effects
- CODAL does not remove the need for good experimental design; experimentalists must include appropriate controls, such as control cells unaffected by the perturbation in both treatment and control batches
- Further improvements require better characterization of technical sources of bias in conjunction with the CODAL model
- The kappa_i softmax denominator is only approximated from the CODAL model, which may introduce slight mismatch between CODAL rates and the augmented NB distribution

## Evidence

- [abstract] CODAL enables the generalization of other count-based generative models to multi-batched data.: "and enables the generalization of other count-based generative models to multi-batched data"
- [intro] MIRA-CODAL architecture relationship: "The CODAL model architecture extends MIRA28, our earlier method for variational topic modeling of single-cell RNA-seq and ATAC-seq data."
- [methods] Gene restriction for RP model: "2841 genes which were both highly variable and had UCSC TSS annotations"
- [methods] Benchmark dataset identity: "details the NEURIPs bone marrow multimodal batch effect benchmark dataset"
- [readme] CODAL availability and role: [evidence span withheld — anchor status: external_doc]
- [methods] Preprocessing thresholds via scanpy: "cell read depths normalized using the normalize_total function with target_sum set to 10000"
