---
name: single-cell-rnaseq-reference-pretraining
description: Use when building a reference-based cell-type annotation model from multiple
  scRNA-seq datasets generated on different sequencing platforms and you need batch-effect-free,
  pre-trained weights for downstream label transfer.
when_to_use_negative:
- Annotating disease query data with the published disease models — the README removes
  the fine-tuning step and recommends users train on their own data since disease
  datasets are extremely diverse across patients and groups
- The query data is already normalized and clustered and only label transfer is needed
  — use 'selina predict' directly instead of re-training
- The reference consists of a single dataset/platform — MADA's K platform-specific
  adversarial classifiers and gradient reversal provide no benefit without multiple
  domains
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SELINA
  role: Framework implementing SMOTE balancing, MADA pre-training with gradient reversal
    layer, and autoencoder fine-tuning ('selina train')
  repo: https://github.com/SELINA-team/SELINA.py
- name: PyTorch
  role: Deep learning backend for the MADA network, gradient reversal layer, and autoencoder
    (pytorch=1.10.0 in the environment)
- name: imbalanced-learn
  role: Provides SMOTE used to oversample minority cell types before pre-training
    (imbalanced-learn=0.8.1)
- name: presto
  role: Fast differential expression/marker discovery used in the SELINA environment
    for cluster marker genes
  repo: https://github.com/immunogenomics/presto
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:71585ecb6f0021ff580d174e101f4e65f7bd23238086646ba5d1a8ec320fced0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SELINA-team/SELINA.py
  repo_url: https://github.com/SELINA-team/SELINA.py
  tools:
  - PyTorch
  - Python
  - SELINA
  - imbalanced-learn
  - presto
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# single-cell-rnaseq-reference-pretraining

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-train a SELINA reference model from paired expression matrices (xx_expr.txt) and annotations (xx_meta.txt) using SMOTE cell-type balancing, MADA multiple-adversarial domain adaptation with gradient reversal to remove sequencing-platform batch effects, and autoencoder fine-tuning that adapts parameters to the query distribution.

## When to use

Use when building a reference-based cell-type annotation model from multiple scRNA-seq datasets generated on different sequencing platforms and you need batch-effect-free, pre-trained weights for downstream label transfer.

## When NOT to use

- Annotating disease query data with the published disease models — the README removes the fine-tuning step and recommends users train on their own data since disease datasets are extremely diverse across patients and groups
- The query data is already normalized and clustered and only label transfer is needed — use 'selina predict' directly instead of re-training
- The reference consists of a single dataset/platform — MADA's K platform-specific adversarial classifiers and gradient reversal provide no benefit without multiple domains

## Inputs

- Reference expression matrices in plain format (xx_expr.txt, e.g. GSE139534_Lung-Adult_10462_expr.txt)
- Reference metadata files with cell-type and platform annotations (xx_meta.txt)
- Query expression matrix (query_expr.txt) for the autoencoder fine-tuning step

## Outputs

- pre-trained_params.pt containing all parameters of the trained model
- pre-trained_meta.pkl containing the cell types and genes of the reference data (plus cell source information in disease condition)

## How to apply

Load all paired reference files (xx_expr.txt expression matrices and xx_meta.txt cell-type/platform annotations) from the reference data directory. Oversample rare cell types with SMOTE (Synthetic Minority Oversampling Technique) so minority cell types are not ignored during modeling. Pre-train the network with MADA: a shared feature extractor feeding a cell-type discriminator and K platform-specific classifiers, where a gradient reversal layer reverses the feature gradient during backward propagation so the feature extractor maximizes the platform discriminator's loss while the discriminator minimizes its own loss, with the balancing loss weight lambda following a sigmoid schedule over 50 epochs. Fine-tune with an autoencoder (encoder = feature extractor plus cell-type discriminator): first train the decoder, then train the encoder to fit the query data distribution, then remove the decoder. Serialize the final weights to pre-trained_params.pt together with reference genes and cell-type keys (e.g. Club, Ciliated, Macrophage for the lung atlas) in pre-trained_meta.pkl.

## Related tools

- **SELINA** (Framework implementing SMOTE balancing, MADA pre-training with gradient reversal layer, and autoencoder fine-tuning ('selina train')) — https://github.com/SELINA-team/SELINA.py
- **PyTorch** (Deep learning backend for the MADA network, gradient reversal layer, and autoencoder (pytorch=1.10.0 in the environment))
- **imbalanced-learn** (Provides SMOTE used to oversample minority cell types before pre-training (imbalanced-learn=0.8.1))
- **presto** (Fast differential expression/marker discovery used in the SELINA environment for cluster marker genes) — https://github.com/immunogenomics/presto

## Examples

```
selina train --path-in ./reference_data/ --path-out ./res/pre-train/ --outprefix pre-trained
```

## Evaluation signals

- The training script runs to completion and prints 'Finish Training'
- Both output artifacts are produced: pre-trained_params.pt and pre-trained_meta.pkl
- pre-trained_meta.pkl exposes the reference genes and cell-type keys (e.g. Club, Ciliated, Macrophage for the lung atlas built from GSE139534, GSE123405, GSE130148)
- The pre-training loop completes the sigmoid-scheduled lambda over 50 epochs and fine-tuning stages run ('Fine-tuning1' 50/50, 'Fine-tuning2' 10/10)
- Platform discriminator loss behavior is consistent with adversarial training: the feature extractor maximizes platform-prediction loss via gradient reversal so features from different platforms become indistinguishable

## Limitations

- The reference still suffers from imbalances in tissue-specific datasets and cell-type abundance despite its large scale
- Disease models are extremely diverse across patients and groups, so the fine-tuning step is removed (via --disease) and users are recommended to use their own data
- Autoencoder fine-tuning adapts parameters to a specific query distribution, so a model fine-tuned on one query should be regenerated for substantially different query data

## Evidence

- [methods] GRL mechanism from the finding on platform-discriminator training: [evidence span withheld — anchor status: not_found]
