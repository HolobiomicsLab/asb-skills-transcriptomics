---
name: grna-gene-association-filtering
description: Use when you have gRNA-gene association tables (TSS-repression and trans-association
  Q-values and logFC) from Normalisr on a high-MOI CRISPRi screen and need to call
  high-confidence putative gene regulations for GRN reconstruction.
when_to_use_negative:
- You need causal network inference via v-structures, regularized regression, or Granger
  causality with longitudinal data — Normalisr explicitly does not support these
- Data are from full-length (non-UMI) scRNA-seq platforms — nonlinear cellular summary
  covariates were only validated on UMI-based datasets
- You only need cis/TSS-targeting efficacy calls without trans-gene GRN reconstruction
  — the multi-step reproducibility and cross-screen filters are unnecessary overhead
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Normalisr
  role: Normalization–association framework whose association module computes the
    gRNA-gene TSS-repression and trans-association results (Q-values, logFC) that
    are then filtered by the specified cutoffs
  repo: github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:3bcf9517c3892d7969659b5d469a1b8ebdbd0057940336062261ce81cf0c2597
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Normalisr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# grna-gene-association-filtering

## Summary

Filters gRNA–gene association results from high-MOI CRISPRi (CROP-seq/Perturb-seq) screens computed with Normalisr's association module, applying TSS-repression and trans-association cutoffs plus reproducibility filters to reconstruct a causal gene regulatory network of 833 high-confidence regulations.

## When to use

Use when you have gRNA-gene association tables (TSS-repression and trans-association Q-values and logFC) from Normalisr on a high-MOI CRISPRi screen and need to call high-confidence putative gene regulations for GRN reconstruction.

## When NOT to use

- You need causal network inference via v-structures, regularized regression, or Granger causality with longitudinal data — Normalisr explicitly does not support these
- Data are from full-length (non-UMI) scRNA-seq platforms — nonlinear cellular summary covariates were only validated on UMI-based datasets
- You only need cis/TSS-targeting efficacy calls without trans-gene GRN reconstruction — the multi-step reproducibility and cross-screen filters are unnecessary overhead

## Inputs

- Full-scale CROP-seq screen gRNA-gene TSS-repression association table (Q-values, logFC) from Normalisr's association module
- Full-scale CROP-seq screen trans-association results table
- Small-scale CROP-seq screen TSS-repression and trans-association results tables
- gRNA assignment and UMI read count matrices (downloaded from GEO)

## Outputs

- Putative gene regulatory network table (supplementary_data_3.tsv) containing 833 high-confidence gRNA → targeted gene → trans-gene regulations, each satisfying all four Q ≤ 0.2

## How to apply

Load the gRNA-gene association tables from the full-scale and small-scale CROP-seq screens computed with Normalisr's association module, using TSS-targeting gRNA presence as an instrumental variable. Confirm gRNA targeting efficacy by calling TSS-repression regulations with Q ≤ 0.05 and logFC ≤ −0.2. Call trans-association regulations with Q ≤ 0.2 and |logFC| ≥ 0.05. Then sequentially exclude candidate regulations showing nearby-gene repression (potential local/cis effects), regulations not reproducible across distinct gRNAs targeting the same regulator, and regulations not reproducible between the full-scale and small-scale screens. Compile the surviving gRNA → targeted gene → trans-gene relationships into the putative gene regulatory network, written as supplementary_data_3.tsv (as visualized in Fig. 6e).

## Related tools

- **Normalisr** (Normalization–association framework whose association module computes the gRNA-gene TSS-repression and trans-association results (Q-values, logFC) that are then filtered by the specified cutoffs) — github.com/lingfeiwang/normalisr

## Evaluation signals

- The final compiled table contains exactly the 833 regulations reported in Supplementary Data 3
- Every surviving regulation satisfies all four Q ≤ 0.2 criteria
- TSS-repression calls meet Q ≤ 0.05 and logFC ≤ −0.2; trans-association calls meet Q ≤ 0.2 and |logFC| ≥ 0.05
- Each retained regulation is reproducible across distinct gRNAs targeting the same regulator
- Each retained regulation is present in both the full-scale and small-scale screens (no cross-screen irreproducibility); candidate regulations with nearby-gene repression are excluded

## Limitations

- Normalisr does not reconstruct causal regulatory networks through v-structures or regularized regression, nor Granger causality with longitudinal information
- Only UMI-based datasets were analyzed; full-length platforms were not validated and may require revalidation of the nonlinear cellular summary covariates
- Batch effects in multi-dataset integrative studies are not addressed and may require additional consideration
- Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations

## Evidence

- [results] 833 high-confidence putative gene regulations: [evidence span withheld — anchor status: not_found]
- [results] TSS-repression thresholds: [evidence span withheld — anchor status: not_found]
- [results] trans-association thresholds: [evidence span withheld — anchor status: not_found]
- [abstract] GRN from trans-effects of gRNAs: "The superior scalability allows us to reconstruct robust gene regulatory networks from trans-effects of guide RNAs in large-scale single cell CRISPRi screens."
- [readme] tsv file format: [evidence span withheld — anchor status: external_doc]
