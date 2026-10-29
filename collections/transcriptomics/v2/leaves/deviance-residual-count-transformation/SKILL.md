---
name: deviance-residual-count-transformation
description: Use when raw UMI count matrices (e.g., scRNA-seq .h5ad objects) are about
  to be encoded by a MIRA/CODAL topic model and a continuous, variance-stabilized
  expression representation is required instead of raw or log-normalized counts.
when_to_use_negative:
- Feeding a downstream topic model that expects raw integer counts or its own native
  normalization (e.g., non-MIRA topic models) — deviance residuals are specific to
  CODAL/MIRA's encoder design
- scATAC-seq accessibility data or non-count modalities where deviance residuals of
  counts are not defined as described in the CODAL RNA-seq workflow
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MIRA
  role: Provides the CODAL expression topic model whose encoder consumes deviance-residual-transformed
    counts (mira.topics.make_model)
  repo: https://github.com/cistrome/MIRA
- name: scanpy
  role: Performs the upstream filtering, normalize_total (target_sum=10000), log+1
    transform, and highly variable gene selection before deviance residual transformation
- name: CODAL
  role: The variational inference procedure whose encoder takes deviance residuals
    as input during batch-disentangling training
  repo: https://github.com/cistrome/MIRA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:85a23c3e814bd29621fd7d48e381c9648d6fa828302a547fe141293663a50cd9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cistrome/MIRA
  tools:
  - CODAL
  - MIRA
  - PyTorch
  - Pyro
  - scanpy
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# deviance-residual-count-transformation

## Summary

Transforms raw single-cell RNA-seq counts into deviance residuals so count data can be fed through the Gaussian-style encoder of the CODAL/MIRA expression topic model, replacing normalization-based inputs for the variational encoder.

## When to use

Use when raw UMI count matrices (e.g., scRNA-seq .h5ad objects) are about to be encoded by a MIRA/CODAL topic model and a continuous, variance-stabilized expression representation is required instead of raw or log-normalized counts.

## When NOT to use

- Feeding a downstream topic model that expects raw integer counts or its own native normalization (e.g., non-MIRA topic models) — deviance residuals are specific to CODAL/MIRA's encoder design
- scATAC-seq accessibility data or non-count modalities where deviance residuals of counts are not defined as described in the CODAL RNA-seq workflow

## Inputs

- Raw scRNA-seq UMI count matrix (AnnData/.h5ad), e.g., NEURIPS bone marrow counts filtered to cells >=400 counts and genes >=30 counts
- Highly variable gene selection (3500 genes, minimum dispersion 0.3) from normalize_total(target_sum=10000) + log+1 preprocessing

## Outputs

- Deviance-residual-transformed expression matrix (Ncells x 3500 genes) used as encoder input to the CODAL topic model

## How to apply

First preprocess the NEURIPS bone marrow scRNA-seq counts with scanpy per the CODAL methods: filter cells with fewer than 400 counts and genes with fewer than 30 counts, run normalize_total with target_sum=10000, log+1 transform, and select 3500 highly variable genes with a minimum dispersion threshold of 0.3. Then, instead of feeding these values directly to the encoder, transform the raw expression counts using deviance residuals, which the CODAL pipeline uses as the encoder input. The deviance-residual transform retains gene-specific mean–variance structure while producing approximately homoscedastic values suitable for the Dirichlet variational encoder instantiated via mira.topics.make_model (e.g., Ntopics=22, total pseudocount 50). The transformed matrix, together with one-hot technical covariates (sequencing site, donor), is what enters the encoder during cyclical stochastic minibatch training.

## Related tools

- **MIRA** (Provides the CODAL expression topic model whose encoder consumes deviance-residual-transformed counts (mira.topics.make_model)) — https://github.com/cistrome/MIRA
- **scanpy** (Performs the upstream filtering, normalize_total (target_sum=10000), log+1 transform, and highly variable gene selection before deviance residual transformation)
- **CODAL** (The variational inference procedure whose encoder takes deviance residuals as input during batch-disentangling training) — https://github.com/cistrome/MIRA

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('neurips_rna.h5ad'); sc.pp.filter_cells(adata, min_counts=400); sc.pp.filter_genes(adata, min_counts=30); sc.pp.normalize_total(adata, target_sum=1e4); sc.pp.log1p(adata); sc.pp.highly_variable_genes(adata, min_disp=0.3)
```

## Evaluation signals

- Pre-filter invariant: cells with fewer than 400 counts and genes with fewer than 30 counts must be removed before transformation
- Output matrix shape must be Ncells x 3500 genes matching the highly variable gene selection (minimum dispersion 0.3)
- Transformed values should be continuous, real-valued residuals (not integer counts), suitable for the Gaussian-style encoder input of the Dirichlet variational model
- Downstream check: the CODAL topic model trained on deviance-residual inputs must reach a converged CODAL objective score on held-out cells

## Limitations

- Deviance residuals are one preprocessing step only; authors caution that batch-confounded cell states inferred downstream should be validated with orthogonal biological evidence to rule out under-correction of technical effects
- Requires the accompanying scanpy preprocessing (filtering, normalize_total target_sum=10000, log+1, HVG selection) to be applied consistently, or the encoder input distribution shifts and training may not converge

## Evidence

- [methods] Deviance residuals are the encoder input: "Raw expression counts are transformed using deviance residuals61 before being fed through the encoder model."
- [methods] Upstream filtering thresholds: "cells with fewer than 400 counts and genes with fewer than 30 counts filtered out"
- [methods] Normalization and HVG selection parameters: "cell read depths normalized using the normalize_total function with target_sum set to 10000"
- [methods] HVG count and dispersion threshold: "highly variable genes found with a minimum dispersion threshold of 0.3, yielding 3500 highly variable genes"
- [readme] All MIRA topic models use CODAL inference: [evidence span withheld — anchor status: external_doc]
