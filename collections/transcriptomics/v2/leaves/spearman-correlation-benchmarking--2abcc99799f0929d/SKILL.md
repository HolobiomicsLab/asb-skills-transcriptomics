---
name: spearman-correlation-benchmarking
description: Use when comparing feature-imputation methods on unpaired multimodal
  single-cell data where held-out features (e.g. smFISH-measured genes) have ground-truth
  spatial or quantitative values to correlate against predictions.
when_to_use_negative:
- When no ground-truth measurements exist for the held-out features, since Spearman
  correlation against imputed values cannot be computed
- When integrating modality pairs without cross-modality feature connections (e.g.
  electrophysiology vs gene expression), for which the underlying imputation pipeline
  is not applicable
edam_operation: http://edamontology.org/operation_0470
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: scConfluence
  role: Method under evaluation; imputes held-out smFISH gene expression from shared
    latent embeddings learned with uncoupled autoencoders and Sinkhorn (rIOT) regularization
  repo: https://github.com/cantinilab/scconfluence
- name: Seurat v3.0
  role: Baseline integration method whose imputed spatial patterns are scored with
    the same Spearman correlation metric
- name: Liger
  role: Baseline integration method benchmarked against scConfluence on the scRNA-smFISH
    task
- name: MultiMAP
  role: Baseline integration method benchmarked against scConfluence on the scRNA-smFISH
    task
- name: Uniport
  role: Baseline integration method benchmarked against scConfluence on the scRNA-smFISH
    task
- name: scGLUE
  role: Baseline integration method benchmarked against scConfluence on the scRNA-smFISH
    task
- name: Scanpy
  role: Used with matplotlib to plot imputed spatial patterns for Scgn, Synpr, and
    Olah
- name: Muon
  role: Provides the MuData object holding the scRNA and smFISH modalities used for
    imputation and evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:a7677c14ca1f4030b182ffa50be1d30fecd23512e2fab0c64d13fce36c13f620
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cantinilab/scconfluence
  repo_url: https://github.com/cantinilab/scconfluence
  tools:
  - Liger
  - MultiMAP
  - Muon
  - Scanpy
  - Seurat
  - Seurat v3.0
  - Uniport
  - scConfluence
  - scGLUE
  edam_operation: http://edamontology.org/operation_0470
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# spearman-correlation-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark spatial gene-expression imputation quality by computing average and median Spearman correlations between imputed and measured smFISH spatial patterns, comparing scConfluence against baselines such as Seurat v3.0, Liger, MultiMAP, Uniport, and scGLUE on the mouse somatosensory cortex scRNA-smFISH integration.

## When to use

Use when comparing feature-imputation methods on unpaired multimodal single-cell data where held-out features (e.g. smFISH-measured genes) have ground-truth spatial or quantitative values to correlate against predictions.

## When NOT to use

- When no ground-truth measurements exist for the held-out features, since Spearman correlation against imputed values cannot be computed
- When integrating modality pairs without cross-modality feature connections (e.g. electrophysiology vs gene expression), for which the underlying imputation pipeline is not applicable

## Inputs

- Imputed spatial expression values for the 11 held-out smFISH gene groups from each method (scConfluence, Seurat v3.0, Liger, MultiMAP, Uniport, scGLUE)
- Measured smFISH spatial expression patterns for the same held-out genes in the mouse somatosensory cortex dataset

## Outputs

- Per-method average and median Spearman correlations between imputed and measured smFISH spatial patterns
- Method ranking (scConfluence vs. five baselines)
- Plots of imputed spatial patterns for Scgn, Synpr, and Olah (Scanpy/matplotlib), comparable to Fig. 4d-f

## How to apply

After training scConfluence (uncoupled autoencoders with Sinkhorn/rIOT regularization, latent dimension 16) and each baseline on the Smartseq2 scRNA-seq and smFISH modalities of a MuData object, use the learned shared latent embeddings to impute expression for the 11 held-out smFISH gene groups. For each method, compute Spearman correlations between imputed and measured smFISH spatial patterns per gene group, then summarize with both the average and the median correlation to obtain a robust ranking. Acceptance requires that the average/median Spearman correlations reproduce the scConfluence-vs-baseline ranking and that imputed spatial patterns for Scgn, Synpr, and Olah match Fig. 4d-f of the paper. Plot imputed spatial patterns with Scanpy/matplotlib to visually validate predicted patterns.

## Related tools

- **scConfluence** (Method under evaluation; imputes held-out smFISH gene expression from shared latent embeddings learned with uncoupled autoencoders and Sinkhorn (rIOT) regularization) — https://github.com/cantinilab/scconfluence
- **Seurat v3.0** (Baseline integration method whose imputed spatial patterns are scored with the same Spearman correlation metric)
- **Liger** (Baseline integration method benchmarked against scConfluence on the scRNA-smFISH task)
- **MultiMAP** (Baseline integration method benchmarked against scConfluence on the scRNA-smFISH task)
- **Uniport** (Baseline integration method benchmarked against scConfluence on the scRNA-smFISH task)
- **scGLUE** (Baseline integration method benchmarked against scConfluence on the scRNA-smFISH task)
- **Scanpy** (Used with matplotlib to plot imputed spatial patterns for Scgn, Synpr, and Olah)
- **Muon** (Provides the MuData object holding the scRNA and smFISH modalities used for imputation and evaluation)

## Examples

```
from scipy.stats import spearmanr; avg_rho = spearmanr(imputed_spatial, measured_spatial).statistic; import scanpy as sc; sc.pl.spatial(adata, color=['Scgn','Synpr','Olah'])
```

## Evaluation signals

- Average and median Spearman correlations reproduce the scConfluence-vs-baseline ranking reported in the paper
- Imputed spatial patterns for Scgn, Synpr, and Olah visually match Fig. 4d-f of the paper
- All 11 held-out smFISH gene groups are imputed and scored for each of the six methods
- Both average and median Spearman correlation are reported (median guards against outlier gene groups dominating the ranking)

## Limitations

- Rare cell populations are hard to detect and under-represented in parameter estimation, especially with mini-batch gradient descent, which can degrade imputation and thus correlation scores
- scConfluence relies on cross-modality feature connections; the benchmark is only applicable where such weakly connected features exist between scRNA genes and smFISH probes
- Hyperparameters such as sinkhorn_loss_weight/reach may need tuning if modality embeddings separate, affecting reproducibility of the correlation ranking

## Evidence

- [other] Compute average and median Spearman correlations between imputed and measured smFISH spatial patterns across all methods: [evidence span withheld — anchor status: not_found]
- [other] Validation acceptance criterion: [evidence span withheld — anchor status: not_found]
