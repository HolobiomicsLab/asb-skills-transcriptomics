---
name: transcriptomic-gene-list-annotation
description: Use when a fitted suffPCR model on transcriptomic data yields feature-sparse
  loadings and the nonzero feature list must be mapped to genes and grouped into functional
  families for interpretation.
when_to_use_negative:
- The feature list is already annotated gene identifiers and only needs counting —
  no sparse-loadings extraction is needed.
- The fitted model has dense (non-sparse) loadings, so there is no natural small subset
  of genes to annotate.
- The analysis goal is univariate differential-expression testing rather than interpreting
  a row-sparse PCA + regression subspace.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: suffpcr
  role: Provides the fitted model and coef.suffpcr() accessor whose nonzero feature-sparse
    loadings are annotated; also bundles the dlbcl expression dataset.
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Runtime environment in which the suffpcr model is fitted, coefficients extracted,
    and the gene-list CSV written.
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Installs the suffpcr package from GitHub so the dlbcl dataset and coef.suffpcr
    are available.
  repo: https://github.com/dajmcdon/suffpcr
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:e3e9b9c230bc7cafe94c36cd7be38fe52d679af66d988fa20d61755ba0fb8e9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dajmcdon/suffpcr
  tools:
  - R
  - remotes
  - suffpcr
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# transcriptomic-gene-list-annotation

## Summary

Annotate the nonzero, feature-sparse loadings extracted from a fitted SuffPCR model with gene identifiers, tally unique genes vs ESTs, and group them into functional categories (e.g. ribosomal-protein genes, MHCII genes) for biological interpretation.

## When to use

Use when a fitted suffPCR model on transcriptomic data yields feature-sparse loadings and the nonzero feature list must be mapped to genes and grouped into functional families for interpretation.

## When NOT to use

- The feature list is already annotated gene identifiers and only needs counting — no sparse-loadings extraction is needed.
- The fitted model has dense (non-sparse) loadings, so there is no natural small subset of genes to annotate.
- The analysis goal is univariate differential-expression testing rather than interpreting a row-sparse PCA + regression subspace.

## Inputs

- Fitted suffPCR model object (refit on all samples at the cross-validation-selected lambda and subspace dimension d)
- Subspace dimension d and component index i for coef.suffpcr extraction
- Dataset feature identifiers (gene IDs / EST accessions) from the transcriptomic expression dataset (e.g. the dlbcl dataset in the suffpcr package)

## Outputs

- Table of selected features annotated with gene identifiers (e.g. dlbcl_selected_features.csv)
- Tally of unique genes vs ESTs (e.g. 87 features = 32 unique genes + 2 ESTs: Hs.22635/ZBTB44, Hs.343870)
- Functional-group counts (e.g. 17 ribosomal-protein genes, 9 MHCII genes) and flagged genes of interest (CORO1A, FEZ1, RAG1, RYK, CXCL5)

## How to apply

After cross-validating the suffPCR lambda grid (lambda_max, lambda_min, lambda_seq) and refitting on all samples at the best lambda and chosen subspace dimension d, extract the nonzero loadings with coef.suffpcr(object, i, d, ...). Map each nonzero feature to a gene identifier, then tally unique genes (e.g. 87 DLBCL features resolve to 32 unique genes and 2 ESTs, Hs.22635/ZBTB44 and Hs.343870). Group the annotated genes into functional families — in the DLBCL case, 17 ribosomal-protein genes and 9 MHCII genes — and flag individually discussed genes such as CORO1A, FEZ1, RAG1, RYK, CXCL5. Write the structured gene-list table to CSV (e.g. dlbcl_selected_features.csv). The rationale is that SuffPCR's subspace is sparse in features, so predictions depend on only a small subset of genes, making this compact list directly interpretable.

## Related tools

- **suffpcr** (Provides the fitted model and coef.suffpcr() accessor whose nonzero feature-sparse loadings are annotated; also bundles the dlbcl expression dataset.) — https://github.com/dajmcdon/suffpcr
- **R** (Runtime environment in which the suffpcr model is fitted, coefficients extracted, and the gene-list CSV written.) — https://github.com/dajmcdon/suffpcr
- **remotes** (Installs the suffpcr package from GitHub so the dlbcl dataset and coef.suffpcr are available.) — https://github.com/dajmcdon/suffpcr

## Examples

```
remotes::install_github("dajmcdon/suffpcr"); library(suffpcr); feats <- coef.suffpcr(object, i, d); write.csv(feats, "dlbcl_selected_features.csv")
```

## Evaluation signals

- Feature-count invariant: the extracted nonzero loadings total 87 features for the DLBCL signature.
- Gene-tally check: 87 features must resolve to 32 unique genes plus exactly 2 ESTs (Hs.22635/ZBTB44, Hs.343870).
- Functional-group counts reproduce 17 ribosomal-protein genes and 9 MHCII genes in the DLBCL selection.
- Flagged genes of interest (CORO1A, FEZ1, RAG1, RYK, CXCL5) appear in the annotated table.
- Output CSV (dlbcl_selected_features.csv) contains only features with nonzero loadings from coef.suffpcr — no zero-loading features should be present.

## Limitations

- Multiple probe features can map to the same gene (87 features vs 32 unique genes in DLBCL), so feature counts overstate gene counts and annotation multiplicity must be reported.
- Some selected features are ESTs (Hs.22635/ZBTB44, Hs.343870) that cannot be confidently assigned to a single gene.
- The selected gene list depends on the cross-validation-selected lambda and the chosen subspace dimension d; different parameter choices yield different annotated lists.
- The article notes existing sparse approaches 'can fail to select the correct features', so the annotated list should not be treated as ground truth without downstream validation.

## Evidence

- [abstract] SuffPCR first estimates sparse principal components and then estimates a linear model on the recovered subspace.: "SuffPCR first estimates sparse principal components and then estimates a linear model on the recovered subspace."
- [abstract] Because the estimated subspace is sparse in the features, the resulting predictions will depend on only a small subset of genes.: "Because the estimated subspace is sparse in the features, the resulting predictions will depend on only a small subset of genes."
- [intro] The goal of suffpcr is to estimate sufficient principal component regression: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression."
- [methods] remotes::install_github: "remotes::install_github("dajmcdon/suffpcr")"
- [readme] suffPCR example call from README: "out <- suffPCR(X, y, d=1:3)"
