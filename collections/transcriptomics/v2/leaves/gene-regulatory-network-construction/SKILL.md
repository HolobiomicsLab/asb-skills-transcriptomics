---
name: gene-regulatory-network-construction
description: Use when analyzing full-scale or small-scale CROP-seq/Perturb-seq CRISPRi
  screens with TSS-repression and trans-association tables from Normalisr, and the
  goal is a causal gRNA->target->trans-gene regulatory network reproducible across
  gRNAs and screens.
when_to_use_negative:
- The goal is reconstructing causal regulatory networks via v-structures, regularized
  regression, or Granger causality with longitudinal data — the article explicitly
  notes Normalisr does not do these
- The input is not a pooled single-cell CRISPR screen with gRNA assignments (e.g.
  plain co-expression or case-control DE data) — use Normalisr's co-expression or
  DE modules instead
- Hypothesis testing of arbitrary nonlinear associations is required — Normalisr is
  not designed for that
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Normalisr
  role: Normalization and association testing framework whose linear-model module
    computes TSS-repression and trans-association results used as the causal (instrumental-variable)
    basis for network reconstruction
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:e57087d9a17316b3164cca2725d44412a56638c48e8eb232c2862132ec4aa25c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - Normalisr
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gene-regulatory-network-construction

## Summary

Reconstructs a putative gene regulatory network from high-MOI single-cell CRISPRi screens (Perturb-seq/CROP-seq K562 data) by applying Normalisr's causal inference framework to chain TSS-targeting gRNA -> targeted gene -> trans-gene regulations, recovering 833 high-confidence edges (all four Q <= 0.2) as in Supplementary Data 3.

## When to use

Use when analyzing full-scale or small-scale CROP-seq/Perturb-seq CRISPRi screens with TSS-repression and trans-association tables from Normalisr, and the goal is a causal gRNA->target->trans-gene regulatory network reproducible across gRNAs and screens.

## When NOT to use

- The goal is reconstructing causal regulatory networks via v-structures, regularized regression, or Granger causality with longitudinal data — the article explicitly notes Normalisr does not do these
- The input is not a pooled single-cell CRISPR screen with gRNA assignments (e.g. plain co-expression or case-control DE data) — use Normalisr's co-expression or DE modules instead
- Hypothesis testing of arbitrary nonlinear associations is required — Normalisr is not designed for that

## Inputs

- UMI read count matrix and gRNA assignment matrix from high-MOI CRISPRi screens (GEO, e.g. GSE120861 full-scale and small-scale CROP-seq K562), sparse mtx or tsv format
- Normalisr TSS-repression association results (gRNA-gene, Q-value and logFC)
- Normalisr trans-association results (gRNA-gene, Q-value and logFC)

## Outputs

- Set of high-confidence putative gene regulations (all four Q <= 0.2) forming the reconstructed gene regulatory network (Supplementary Data 3; written as supplementary_data_3.tsv)
- Network edge table (regulator gRNA, targeted gene, trans-gene) for visualization as in Fig. 6e

## How to apply

Run Normalisr's association module on the full-scale and small-scale screen UMI count and gRNA assignment matrices (e.g. downloaded from GEO, GSE120861 highmoi for CRISPRi) to produce gRNA-gene association tables for TSS-repression and trans-association. Call TSS-repression regulations with Q <= 0.05 and logFC <= -0.2 to confirm gRNA targeting efficacy, then call trans-association regulations with Q <= 0.2 and |logFC| >= 0.05. Chain the relationships as TSS-targeting gRNA -> targeted gene -> trans-gene, using gRNA presence as an instrumental variable, and compile surviving edges into a network table (supplementary_data_3.tsv) for visualization as in Fig. 6e. Rationale: Normalisr's normalization-association linear-model framework gives exact P-values without permutation and natively removes covariates (e.g. untested gRNAs) as fixed effects, enabling causal inference at K562 screen scale where SCEPTRE and scMAGECK take weeks and do not account for off-target effects.

## Related tools

- **Normalisr** (Normalization and association testing framework whose linear-model module computes TSS-repression and trans-association results used as the causal (instrumental-variable) basis for network reconstruction) — https://github.com/lingfeiwang/normalisr

## Examples

```
normalisr de -h  # inspect Normalisr's differential-expression/screen submodule used for gRNA-gene association testing
```

## Evaluation signals

- The final table contains exactly the 833 regulations reported in Supplementary Data 3
- Every retained regulation satisfies all four significance criteria with Q <= 0.2 (TSS-repression: Q <= 0.05 and logFC <= -0.2; trans-association: Q <= 0.2 and |logFC| >= 0.05)
- Regulations showing nearby-gene repression (potential local/cis effects) have been excluded
- Each retained regulation is supported by distinct gRNAs targeting the same regulator (gRNA-level reproducibility)
- Each retained regulation reproduces in both the full-scale and small-scale CROP-seq screens (cross-screen irreproducibility removed)

## Limitations

- Normalisr does not reconstruct causal regulatory networks through v-structures or regularized regression, nor search for Granger causality with longitudinal information
- Existing alternatives (e.g. scMAGECK and SCEPTRE) take weeks or more on a modern dataset and do not account for off-target effects
- QC restrictions apply: cells require at least 500 reads and 100 genes with non-zero reads; robustness against read count distortions requires > 100 cells

## Evidence

- [abstract] Normalisr enables reconstruction of robust gene regulatory networks from trans-effects of guide RNAs in large-scale single cell CRISPRi screens: [evidence span withheld — anchor status: not_found]
- [intro] existing CRISPR-screen methods are slow and off-target unaware: "existing methods (e.g., scMageck12 and SCEPTRE13) take weeks or more on a modern dataset and do not account for off-target effects"
- [results] highmoi demonstration subset: "Only a subset of 30 gRNAs and ~7000 cells are extracted from GSE120861 highmoi dataset for this demonstration."
- [readme] Normalisr two-step framework: "Normalisr first removes confounding technical noises from raw read counts to recover the biological variations."
- [readme] association step natively removes covariates such as untested gRNAs: "(ii) native removal of covariates (*e.g.* batches, house-keeping programs, and untested gRNAs) as fixed effects"
