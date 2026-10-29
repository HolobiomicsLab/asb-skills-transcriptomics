---
name: cross-screen-reproducibility-filtering
description: Use when reconstructing a causal gene regulatory network from gRNA-to-gene
  perturbation data where multiple gRNAs per regulator and replicate screens are available,
  and only regulations surviving all four Q ≤ 0.2 tests should be kept.
when_to_use_negative:
- The goal is testing arbitrary nonlinear associations or reconstructing causal networks
  via v-structures, regularized regression, or Granger causality — Normalisr explicitly
  does not support these
- The study spans multiple datasets requiring batch-effect correction, which Normalisr's
  framework does not address
- The data are from full-length (non-UMI) scRNA-seq platforms, for which Normalisr's
  nonlinear covariates were not validated
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Normalisr
  role: Computes the TSS-repression and trans-association tests (gRNA presence as
    instrumental variable) whose Q-values and logFC feed the cross-screen reproducibility
    filter
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:37baacef629503e64110875bf3d6f2c025994223b699c3a71c57d5fa48a08aed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - Normalisr
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-screen-reproducibility-filtering

## Summary

A causal-filtering procedure that compiles high-confidence putative gene regulatory networks (833 regulations) from high-MOI CRISPRi screens by intersecting Normalisr's TSS-repression and trans-association calls across gRNAs and across the full-scale and small-scale CROP-seq screens.

## When to use

Use when reconstructing a causal gene regulatory network from gRNA-to-gene perturbation data where multiple gRNAs per regulator and replicate screens are available, and only regulations surviving all four Q ≤ 0.2 tests should be kept.

## When NOT to use

- The goal is testing arbitrary nonlinear associations or reconstructing causal networks via v-structures, regularized regression, or Granger causality — Normalisr explicitly does not support these
- The study spans multiple datasets requiring batch-effect correction, which Normalisr's framework does not address
- The data are from full-length (non-UMI) scRNA-seq platforms, for which Normalisr's nonlinear covariates were not validated

## Inputs

- Full-scale CROP-seq screen gRNA–gene association table (Normalisr TSS-repression results)
- Small-scale CROP-seq screen gRNA–gene association table (Normalisr TSS-repression results)
- Full-scale trans-association results (gRNA presence as instrumental variable, computed with Normalisr)
- Small-scale trans-association results
- gRNA-to-targeted-gene mapping and nearby-gene annotation

## Outputs

- supplementary_data_3.tsv: the reconstructed putative gene regulatory network of 833 high-confidence regulations, visualized as in Fig. 6e

## How to apply

Load the full-scale and small-scale screen gRNA–gene association tables produced by Normalisr's association module (TSS-repression and trans-association results). Call TSS-repression regulations with Q ≤ 0.05 and logFC ≤ −0.2 to confirm gRNA targeting efficacy, then call trans-association regulations with Q ≤ 0.2 and |logFC| ≥ 0.05. Exclude candidate regulations showing nearby-gene repression (potential local/cis effects) and regulations not reproducible across distinct gRNAs targeting the same regulator. Finally, retain only regulations reproducible between the full-scale and small-scale screens to remove cross-screen irreproducibility, and compile survivors into the network. The rationale is that requiring concordance across both independent screens and multiple gRNAs, with all four Q-values ≤ 0.2, filters spurious trans-associations and off-target effects.

## Related tools

- **Normalisr** (Computes the TSS-repression and trans-association tests (gRNA presence as instrumental variable) whose Q-values and logFC feed the cross-screen reproducibility filter) — https://github.com/lingfeiwang/normalisr

## Examples

```
normalisr de -h
```

## Evaluation signals

- The final supplementary_data_3.tsv contains exactly the 833 high-confidence putative gene regulations
- Every retained regulation satisfies all four Q ≤ 0.2 tests
- TSS-repression calls obey Q ≤ 0.05 and logFC ≤ −0.2; trans-association calls obey Q ≤ 0.2 and |logFC| ≥ 0.05
- No surviving regulation shows nearby-gene repression or fails to reproduce across distinct gRNAs targeting the same regulator
- Each regulation is reproduced in both the full-scale and small-scale CROP-seq screens

## Limitations

- Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations
- Normalisr does not reconstruct causal regulatory networks via v-structures or regularized regression, nor Granger causality with longitudinal data
- Integrative studies spanning multiple datasets may require additional consideration of batch effects
- Only UMI-based datasets were analyzed; full-length platforms were not validated and may need revalidation of the nonlinear cellular summary covariates

## Evidence

- [results] 833 high-confidence regulations from full-scale and small-scale screens: [evidence span withheld — anchor status: not_found]
- [results] TSS-repression thresholds: [evidence span withheld — anchor status: not_found]
- [abstract] robust GRN from trans-effects: [evidence span withheld — anchor status: not_found]
- [readme] tsv format for I/O: [evidence span withheld — anchor status: not_found]
