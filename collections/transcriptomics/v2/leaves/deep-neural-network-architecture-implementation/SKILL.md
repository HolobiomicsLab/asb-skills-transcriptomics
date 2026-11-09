---
name: deep-neural-network-architecture-implementation
description: Use when building a reference-based single-cell annotation model that
  must remove batch effects across multiple sequencing platforms in a large reference
  atlas before transferring labels to a query dataset.
when_to_use_negative:
- Predicting disease datasets where expression profiles deviate substantially from
  normal data — the fine-tuning step should be removed (--disease) rather than adapted
- Annotating with an already-published SELINA pre-trained model from SELINA-team/SELINA-reference
  instead of training a new network on your own reference data
- Cell-type abundance is already balanced and single-platform reference data is used,
  removing the need for SMOTE and the multi-adversarial domain adaptation design
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: SELINA
  role: Framework whose MADA pre-training network and autoencoder fine-tuning step
    are implemented and trained
  repo: https://github.com/SELINA-team/SELINA.py
- name: Python
  role: Implementation language for the reference-construction and training pipeline
- name: PyTorch
  role: Deep learning library used to implement the feature extractor, gradient reversal
    layer, platform classifiers, and autoencoder
- name: SELINA-reference
  role: Repository of pre-trained models produced by this architecture, covering 35
    normal tissues and three disease models
  repo: https://github.com/SELINA-team/SELINA-reference
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:bc6dda9e5aff2ce2025996e2f2f212d74bbd53f98a55846fa9f0ca8e828321c8
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
  - SELINA-reference
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# deep-neural-network-architecture-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implement and train the deep neural network architecture underlying SELINA: a shared feature extractor with MADA (multi-adversarial domain adaptation) pre-training on multi-platform reference data, followed by autoencoder fine-tuning on query data, implemented in Python/PyTorch and serialized as pre-trained_params.pt with metadata in pre-trained_meta.pkl.

## When to use

Use when building a reference-based single-cell annotation model that must remove batch effects across multiple sequencing platforms in a large reference atlas before transferring labels to a query dataset.

## When NOT to use

- Predicting disease datasets where expression profiles deviate substantially from normal data — the fine-tuning step should be removed (--disease) rather than adapted
- Annotating with an already-published SELINA pre-trained model from SELINA-team/SELINA-reference instead of training a new network on your own reference data
- Cell-type abundance is already balanced and single-platform reference data is used, removing the need for SMOTE and the multi-adversarial domain adaptation design

## Inputs

- Paired reference files: xx_expr.txt gene-by-cell expression matrices and xx_meta.txt cell-type/platform annotations from the reference data directory
- Query expression matrix (query_expr.txt, plain/h5/mtx formats supported) from selina preprocess
- Reference data such as GSE139534, GSE123405, GSE130148 lung-adult datasets

## Outputs

- pre-trained_params.pt: serialized model weights (feature extractor plus cell-type discriminator after decoder removal)
- pre-trained_meta.pkl: pickle containing the reference genes, cell-type keys (e.g. Club, Ciliated, Macrophage), and cell source info for disease condition

## How to apply

Load all paired reference files (xx_expr.txt expression matrices and xx_meta.txt cell-type/platform annotations) and balance cell-type abundances by oversampling minority cell types with SMOTE (imbalanced-learn) before training. Pre-train a network of a shared feature extractor feeding a cell-type discriminator and K platform-specific classifiers for 50 epochs, using a gradient reversal layer (GRL) so the feature extractor maximizes the platform discriminator's loss while each platform classifier minimizes its own loss, plus a balancing loss whose trade-off weight lambda follows a sigmoid schedule. Fine-tune the pre-trained parameters on query data with an autoencoder whose encoder is the feature extractor plus cell-type discriminator: first train the decoder, then train the encoder to fit the query distribution (Fine-tuning1 50 iterations, Fine-tuning2 10 iterations). Finally remove the decoder and serialize the remaining encoder weights to pre-trained_params.pt along with the reference cell types and genes in pre-trained_meta.pkl; skip the fine-tuning step entirely for disease data (--disease flag) since disease expression profiles deviate from normal data.

## Related tools

- **SELINA** (Framework whose MADA pre-training network and autoencoder fine-tuning step are implemented and trained) — https://github.com/SELINA-team/SELINA.py
- **Python** (Implementation language for the reference-construction and training pipeline)
- **PyTorch** (Deep learning library used to implement the feature extractor, gradient reversal layer, platform classifiers, and autoencoder)
- **SELINA-reference** (Repository of pre-trained models produced by this architecture, covering 35 normal tissues and three disease models) — https://github.com/SELINA-team/SELINA-reference

## Examples

```
selina train --path-in ./reference/normal_data --path-out ./res/
```

## Evaluation signals

- The training script runs to completion and prints 'Finish Training'
- Both pre-trained_params.pt and pre-trained_meta.pkl are produced in the output directory
- pre-trained_meta.pkl exposes the reference genes and cell-type keys (e.g. Club, Ciliated, Macrophage for the lung atlas)
- Fine-tuning progress bars complete at the expected iteration counts (Fine-tuning1 50/50, Fine-tuning2 10/10)
- Platform-specific features become indistinguishable after pre-training, as driven by the GRL reversing gradients of platform-predicting errors

## Limitations

- Cell-type abundance and dataset counts across tissues in the reference remain imbalanced despite its large scale (1.7 million cells, 230 cell types)
- Disease datasets are extremely diverse across patients and groups, so disease models only cover non-small-cell lung carcinoma, type 2 diabetes, and Alzheimer's disease, and users are recommended to train on their own disease data
- Fine-tuning must be skipped (--disease) for disease data because expression profiles deviate substantially from normal data
- Graph-based algorithms for structural information extraction and explainability are planned but not yet implemented

## Evidence

- [methods] During backward propagation of platform-predicting errors, the gradient of the feature vector is reversed, so the feature extractor is trained to maximize the platform discriminator's loss: [evidence span withheld — anchor status: not_found]
- [methods] Pre-train the network with MADA: a shared feature extractor feeding a cell-type discriminator and K platform-specific classifiers, with gradient reversal layer (GRL): [evidence span withheld — anchor status: not_found]
- [readme] Since the expression profiles of disease data may deviate a lot from normal data, we removed the fine-tuning step when predicting for the disease data: [evidence span withheld — anchor status: external_doc]
