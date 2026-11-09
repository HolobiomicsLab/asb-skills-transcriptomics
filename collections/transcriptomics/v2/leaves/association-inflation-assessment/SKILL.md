---
name: association-inflation-assessment
description: Use when analyzing high-MOI single-cell CRISPR screen data (e.g., CROP-seq
  with thousands of gRNAs across >10,000 genes) and you need to assess whether gRNA
  cross-associations inflate FPRs in naive differential expression testing.
when_to_use_negative:
- The analysis goal is hypothesis testing of arbitrary nonlinear associations — Normalisr
  is not designed for this
- The task involves integrating multiple datasets where batch effects are a primary
  concern — Normalisr does not address batch effects in multi-dataset integrative
  studies
- The input data comes from a full-length (non-UMI) scRNA-seq platform — only UMI-based
  datasets were validated, and the nonlinear cellular summary covariates may need
  revalidation
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0085
tools:
- name: Normalisr
  role: Performs QC, normalization (lcpm, normcov, scaling_factor, compute_var, normvar),
    and both naive and competition-aware differential expression ('de' submodule)
    runs; native removal of untested gRNAs as fixed-effect covariates
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:dfc0cff91d771860134d6d367eda5b6342600c7b1328d41bb9345a5d11f3461c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - Normalisr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# association-inflation-assessment

## Summary

Quantifies the false positive rate inflation of gRNA–gene associations in high-MOI single-cell CRISPR screens when untested gRNAs are ignored (competition-naive DE), by comparing naive and competition-aware Normalisr runs via Storey's method on non-targeting-control P-value histograms.

## When to use

Use when analyzing high-MOI single-cell CRISPR screen data (e.g., CROP-seq with thousands of gRNAs across >10,000 genes) and you need to assess whether gRNA cross-associations inflate FPRs in naive differential expression testing.

## When NOT to use

- The analysis goal is hypothesis testing of arbitrary nonlinear associations — Normalisr is not designed for this
- The task involves integrating multiple datasets where batch effects are a primary concern — Normalisr does not address batch effects in multi-dataset integrative studies
- The input data comes from a full-length (non-UMI) scRNA-seq platform — only UMI-based datasets were validated, and the nonlinear cellular summary covariates may need revalidation

## Inputs

- UMI read count matrix (tsv or sparse mtx/Cell Ranger format, gzipped accepted)
- gRNA assignment matrix (tsv) for the pooled CRISPR screen
- Row/column name text files (cells and genes/gRNAs, one per line)
- List of non-targeting-control (NTC) gRNAs for FPR estimation

## Outputs

- gRNA-gene association table with log fold-changes, P values, and Q values for both naive and competition-aware runs (de_associations.tsv)
- Overall FPR estimates among NTCs via Storey's method (~4.8% naive vs ~1.5% competition-aware expected)
- Association-inflation ratio at Q<=0.2 comparing the two configurations

## How to apply

Obtain the UMI read count matrix and gRNA assignment matrix (e.g., the 207,324-cell K562 full-scale CRISPRi screen with 13,186 gRNAs and 10,877 genes from GEO) and preprocess with Normalisr's qc_reads/qc_outlier and lcpm/normcov/scaling_factor/compute_var/normvar normalization to remove nonlinear library-size confounders at mean and variance levels. Run the Normalisr 'de' submodule twice: a naive configuration across all gRNA-gene pairs with no untested-gRNA covariates, and a competition-aware configuration that includes untested gRNAs as fixed-effect covariates, reduced to the top-500 principal components when their count exceeds 10,000. Extract log fold-changes, P values, and Q values for every gRNA-gene pair from both runs. Estimate false positive rates with Storey's method on the P-value histograms of non-targeting-control (NTC) gRNAs — the reference result is ~4.8% overall naive FPR vs ~1.5% competition-aware, and ~10.3% naive among TSS-targeted genes — and compute the association-inflation ratio at Q<=0.2. Write the complete association table with logFCs and Q values to de_associations.tsv.

## Related tools

- **Normalisr** (Performs QC, normalization (lcpm, normcov, scaling_factor, compute_var, normvar), and both naive and competition-aware differential expression ('de' submodule) runs; native removal of untested gRNAs as fixed-effect covariates) — https://github.com/lingfeiwang/normalisr

## Examples

```
normalisr de -h
```

## Evaluation signals

- Storey's-method FPR estimate on NTC P-value histograms falls near the reported values: ~4.8% overall for the naive run and ~1.5% for the competition-aware run (~10.3% naive among TSS-targeted genes)
- Association-inflation ratio at Q<=0.2 is computable and shows the competition-aware run reduces apparent associations relative to the naive run
- de_associations.tsv contains log fold-changes and Q values for every gRNA-gene pair (e.g., 13,186 gRNAs × 10,877 genes for the K562 full-scale screen)
- Competition-aware run reduces untested gRNAs to the top-500 principal components when their number exceeds 10,000 — verify the covariate dimension in the model design
- P-value histograms of NTC gRNAs in the naive run show the characteristic inflation relative to the competition-aware run

## Limitations

- FPR comparison with SCEPTRE and scMAGECK was limited: SCEPTRE P values for non-cis-effects (including NTCs) were unavailable, and scMAGECK did not finish within 2 weeks and its P values were not publicly available
- Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations
- Batch effects in multi-dataset integrative studies are not addressed
- Only UMI-based datasets were analyzed; full-length platforms may require revalidation of the nonlinear cellular summary covariates
- Normalisr does not reconstruct causal regulatory networks via v-structures, regularized regression, or Granger causality

## Evidence

- [other] On the full-scale CROP-seq screen, a competition-naive DE analysis using Normalisr that disregarded other, untested gRNAs led to a 4.8% overall FPR among NTCs (estimated with Storey's method): [evidence span withheld — anchor status: not_found]
- [readme] Native removal of untested gRNAs as fixed effects in Normalisr linear models: "(ii) native removal of covariates (*e.g.* batches, house-keeping programs, and untested gRNAs) as fixed effects"
- [abstract] Normalisr systematically detects and removes nonlinear library-size confounders at mean and variance levels: "By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels, Normalisr achieves high sensitivity, specificity, speed, and generalizability"
- [methods] Normalization functions used to preprocess the count matrix before association testing: "lcpm
	normcov
	scaling_factor
	compute_var
	normvar"
- [abstract] Normalisr scalability motivated the full-scale K562 CRISPRi screen analysis: "The superior scalability allows us to reconstruct robust gene regulatory networks from trans-effects of guide RNAs in large-scale single cell CRISPRi screens."
