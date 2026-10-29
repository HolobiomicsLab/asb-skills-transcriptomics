---
name: crispr-screen-trans-regulatory-inference
description: Use when you have gRNA assignment and UMI count matrices from a high-MOI
  CRISPRi screen (e.g., GSE120861 K562) and want to infer causal trans-gene regulations
  from TSS-targeting gRNAs.
when_to_use_negative:
- You need arbitrary nonlinear association testing — Normalisr is not designed for
  hypothesis testing of nonlinear associations
- You need causal inference via v-structures, regularized regression, or Granger causality
  with longitudinal data — explicitly out of scope
- Data are non-UMI full-length platforms or multi-dataset integrations needing batch-effect
  correction — only UMI-based datasets were validated
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Normalisr
  role: Normalization (lcpm, normcov, compute_var, normvar) and linear-model association
    testing used to call gRNA-gene TSS-repression and trans-association regulations
    for network reconstruction
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:3f82e24d82070dcde34248d88028bc26d9922be77582b1306c7a439806d276c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - Normalisr
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# crispr-screen-trans-regulatory-inference

## Summary

Reconstruct a putative gene regulatory network from high-MOI single-cell CRISPRi (CROP-seq) screens by chaining TSS-repression gRNA→targeted-gene links with trans-associations through Normalisr's causal inference framework, yielding 833 high-confidence regulations (all four Q ≤ 0.2).

## When to use

Use when you have gRNA assignment and UMI count matrices from a high-MOI CRISPRi screen (e.g., GSE120861 K562) and want to infer causal trans-gene regulations from TSS-targeting gRNAs.

## When NOT to use

- You need arbitrary nonlinear association testing — Normalisr is not designed for hypothesis testing of nonlinear associations
- You need causal inference via v-structures, regularized regression, or Granger causality with longitudinal data — explicitly out of scope
- Data are non-UMI full-length platforms or multi-dataset integrations needing batch-effect correction — only UMI-based datasets were validated

## Inputs

- UMI read count matrix from high-MOI CRISPRi CROP-seq screens (e.g., GSE120861 highmoi K562 dataset, full-scale and small-scale variants)
- gRNA assignment matrix (guide-by-cell) for TSS-targeting gRNAs
- gRNA-to-targeted-gene mapping (TSS-targeting design)

## Outputs

- TSS-repression regulation table (gRNA → targeted gene, Q ≤ 0.05, logFC ≤ −0.2)
- Trans-association regulation table (Q ≤ 0.2, |logFC| ≥ 0.05)
- Reconstructed putative gene regulatory network of 833 high-confidence regulations, exported as supplementary_data_3.tsv for visualization as in Fig. 6e

## How to apply

Run Normalisr's association module on the full-scale and small-scale screen UMI/gRNA matrices to produce gRNA-gene association tables (TSS-repression and trans-association results). Call TSS-repression regulations with Q ≤ 0.05 and logFC ≤ −0.2 to confirm gRNA targeting efficacy, then call trans-association regulations with Q ≤ 0.2 and |logFC| ≥ 0.05. Exclude candidate regulations showing nearby-gene repression (potential local/cis effects) and those not reproducible across distinct gRNAs targeting the same regulator, then exclude regulations not reproducible between the full-scale and small-scale screens to remove cross-screen irreproducibility. Compile surviving regulations into the network (supplementary_data_3.tsv), using gRNA presence as an instrumental variable for the TSS-targeted gene's effect on trans-genes. The linear-model association step natively removes untested gRNAs as fixed-effect covariates to control off-target and confounding effects.

## Related tools

- **Normalisr** (Normalization (lcpm, normcov, compute_var, normvar) and linear-model association testing used to call gRNA-gene TSS-repression and trans-association regulations for network reconstruction) — https://github.com/lingfeiwang/normalisr

## Examples

```
pip install normalisr; normalisr -h
```

## Evaluation signals

- Final network table contains exactly 833 high-confidence regulations, each satisfying all four Q ≤ 0.2 criteria
- Every TSS-repression call meets Q ≤ 0.05 and logFC ≤ −0.2; every trans-association call meets Q ≤ 0.2 and |logFC| ≥ 0.05
- No surviving regulation is a nearby-gene repression (local/cis effect) and each is reproduced across distinct gRNAs targeting the same regulator
- Each regulation is reproducible between the full-scale and small-scale CROP-seq screens
- Output TSV matches Supplementary Data 3 and can be rendered as in Fig. 6e

## Limitations

- Normalisr does not reconstruct causal regulatory networks through v-structures or regularized regression, nor Granger causality with longitudinal information
- Regulations are putative; only UMI-based datasets were analyzed and full-length platforms were not validated
- Existing CRISPR screen methods (e.g., scMAGECK, SCEPTRE) were not fully comparable for sensitivity/specificity due to runtime and unavailable P values

## Evidence

- [abstract] The superior scalability allows us to reconstruct robust gene regulatory networks from trans-effects of guide RNAs in large-scale single cell CRISPRi screens.: "The superior scalability allows us to reconstruct robust gene regulatory networks from trans-effects of guide RNAs in large-scale single cell CRISPRi screens."
- [intro] Association step unifies CRISPR screen analysis with linear models: "The association step utilizes linear models to unify case–control DE, single-cell CRISPR screen analysis as multivariate DE, and gene co-expression network inference"
- [readme] Native covariate removal as fixed effects: "native removal of covariates (*e.g.* batches, house-keeping programs, and untested gRNAs) as fixed effects"
- [results] High-MOI example dataset extraction: "Only a subset of 30 gRNAs and ~7000 cells are extracted from GSE120861 highmoi dataset for this demonstration."
