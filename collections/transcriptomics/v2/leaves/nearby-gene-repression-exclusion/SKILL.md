---
name: nearby-gene-repression-exclusion
description: Use when assembling a putative trans-gene regulatory network from TSS-targeting
  gRNA association results in a high-MOI single-cell CRISPRi screen, and candidate
  regulations must be purged of nearby-gene repression signals that likely reflect
  local cis effects rather than causal trans regulation.
when_to_use_negative:
- The goal is to reconstruct causal networks via v-structures, regularized regression,
  or Granger causality with longitudinal data — Normalisr explicitly does not support
  these.
- The dataset is from a full-length (non-UMI) scRNA-seq platform, which the paper
  did not validate.
- The question is hypothesis testing of arbitrary nonlinear associations, for which
  Normalisr is not designed.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Normalisr
  role: Provides the normalization and linear-model association testing (TSS-repression
    and trans-association calls with Q and logFC cutoffs) whose output candidate regulations
    are then filtered for nearby-gene repression before network compilation.
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:c46bcb39ac16b2d837c8633ad36093f1e8c436a543c1b94ea4332b56f3c8328b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - Normalisr
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Nearby-gene repression exclusion

## Summary

A filtering step in Normalisr-based CRISPRi causal inference that removes candidate gRNA→gene regulations whose apparent trans-effect is actually nearby-gene repression (potential local/cis effects), so that only genuine trans-regulations survive into the reconstructed gene regulatory network.

## When to use

Use when assembling a putative trans-gene regulatory network from TSS-targeting gRNA association results in a high-MOI single-cell CRISPRi screen, and candidate regulations must be purged of nearby-gene repression signals that likely reflect local cis effects rather than causal trans regulation.

## When NOT to use

- The goal is to reconstruct causal networks via v-structures, regularized regression, or Granger causality with longitudinal data — Normalisr explicitly does not support these.
- The dataset is from a full-length (non-UMI) scRNA-seq platform, which the paper did not validate.
- The question is hypothesis testing of arbitrary nonlinear associations, for which Normalisr is not designed.

## Inputs

- TSS-repression (gRNA-gene) association table from Normalisr's association module for the full-scale CROP-seq K562 screen
- TSS-repression association table for the small-scale CROP-seq screen
- Trans-association (gRNA-gene) result tables for full-scale and small-scale screens
- Genomic coordinates/annotations of gRNA TSS targets and candidate trans-genes (to identify nearby-gene repression)

## Outputs

- Filtered list of high-confidence putative gene regulations with nearby-gene (local/cis) repression events removed
- Reconstructed gene regulatory network written as supplementary_data_3.tsv (833 regulations, all four Q ≤ 0.2)

## How to apply

First call TSS-repression regulations with Normalisr's association module using Q ≤ 0.05 and logFC ≤ −0.2 to confirm gRNA targeting efficacy, and trans-association regulations using Q ≤ 0.2 and |logFC| ≥ 0.05 in both the full-scale and small-scale CROP-seq (K562) screens. Then scan the candidate gRNA → targeted gene → trans-gene triples and exclude any regulation whose target gene lies in the genomic neighborhood of the TSS-targeted regulator, since repression of genes adjacent to the gRNA cut site is a known local/cis artifact rather than evidence of a causal trans-edge. Reinforce the filter with the two reproducibility gates from the same workflow: require the regulation to be supported by distinct gRNAs targeting the same regulator, and require it to be reproduced between the full-scale and small-scale screens. The surviving high-confidence regulations (all four Q ≤ 0.2) are compiled into the putative gene regulatory network, yielding the 833 regulations reported in Supplementary Data 3.

## Related tools

- **Normalisr** (Provides the normalization and linear-model association testing (TSS-repression and trans-association calls with Q and logFC cutoffs) whose output candidate regulations are then filtered for nearby-gene repression before network compilation.) — https://github.com/lingfeiwang/normalisr

## Evaluation signals

- Every retained regulation passes all four significance tests with Q ≤ 0.2, and TSS-repression calls satisfy Q ≤ 0.05 with logFC ≤ −0.2, trans-association calls Q ≤ 0.2 with |logFC| ≥ 0.05.
- No retained regulation has its target gene in the genomic neighborhood of the TSS-targeted regulator (nearby-gene repression candidates are all excluded).
- Each retained regulation is reproduced across distinct gRNAs targeting the same regulator and between the full-scale and small-scale screens.
- The final table contains exactly the 833 high-confidence regulations reported in Supplementary Data 3.
- Output file supplementary_data_3.tsv renders the network as in Fig. 6e.

## Limitations

- Excluding nearby-gene repression discards potentially real local regulatory signal; only trans-effects far from the gRNA target are kept.
- Normalisr's framework only analyzed UMI-based datasets; full-length platforms may produce different read-count distributions and were not validated.
- Integrative studies spanning multiple datasets may require additional consideration of batch effects, which the framework does not address.

## Evidence

- [abstract] The superior scalability allows us to reconstruct robust gene regulatory networks from trans-effects of guide RNAs in large-scale single cell CRISPRi screens.: "The superior scalability allows us to reconstruct robust gene regulatory networks from trans-effects of guide RNAs in large-scale single cell CRISPRi screens."
- [discussion] Normalisr does not reconstruct causal regulatory networks via v-structures or regularized regression, nor Granger causality with longitudinal data: "reconstructing causal regulatory networks through v-structures or regularized regression55, or searching for Granger causality with longitudinal information4"
- [readme] Native removal of covariates as fixed effects in association testing: "native removal of covariates (*e.g.* batches, house-keeping programs, and untested gRNAs) as fixed effects"
