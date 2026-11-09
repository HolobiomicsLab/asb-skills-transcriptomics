---
name: library-size-offset-estimation
description: Use when fitting per-gene/per-feature GLMs (e.g., Tweedie CPLM/ZICP)
  to raw scRNA-seq counts or semi-continuous omics data with unequal sequencing depth,
  so that differences in depth are absorbed by an offset log(S) rather than by transforming
  counts.
when_to_use_negative:
- Input is already normalized or transformed (e.g., log-CPM, TPM, relative abundances
  that already absorbed depth) — the offset would double-correct for sequencing depth
- Zero-inflation is a property of only a subset of genes — the paper notes Tweedieverse
  is not architected for gene-specific zero-inflation affecting a gene subset
- Data are continuous quantities unrelated to sequencing depth where a size-factor
  offset has no sampling-depth interpretation
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SCRAN
  role: R/Bioconductor package used to estimate cell-specific library sizes that are
    incorporated as offsets in per-gene Tweedie models
- name: Tweedieverse
  role: Differential analysis framework that accepts the log library size as an offset
    in the log-link GLM (CPLM/ZICP)
  repo: github.com/himelmallick/Tweedieverse
- name: cplm (R package)
  role: Fits the Compound Poisson Linear Models whose linear predictor includes log(S)
    as an offset
  repo: github.com/himelmallick/Tweedieverse
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:a24f474b8d13d5791d4453fbcf6597da35d0c2a81a1e8df2edd28aad7f910d26
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bioconductor
  - R
  - SCRAN
  - Tweedieverse
  - cplm
  - cplm (R package)
  - statmod
  - statmod (R package)
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Library-size offset estimation for differential analysis

## Summary

Estimates per-cell (or per-sample) library sizes/normalization factors and enters them as a log-scale offset in per-feature regression models, avoiding ad hoc normalization or transformation prior to differential testing.

## When to use

Use when fitting per-gene/per-feature GLMs (e.g., Tweedie CPLM/ZICP) to raw scRNA-seq counts or semi-continuous omics data with unequal sequencing depth, so that differences in depth are absorbed by an offset log(S) rather than by transforming counts.

## When NOT to use

- Input is already normalized or transformed (e.g., log-CPM, TPM, relative abundances that already absorbed depth) — the offset would double-correct for sequencing depth
- Zero-inflation is a property of only a subset of genes — the paper notes Tweedieverse is not architected for gene-specific zero-inflation affecting a gene subset
- Data are continuous quantities unrelated to sequencing depth where a size-factor offset has no sampling-depth interpretation

## Inputs

- Raw scRNA-seq UMI or non-UMI count matrix (genes x cells) or semi-continuous omics count/relative-abundance matrix
- Sample/cell metadata (e.g., binary group indicator) for the design matrix
- Library sizes or normalization factors (e.g., SCRAN-estimated cell-specific sizes)

## Outputs

- Per-gene/per-feature model fits with log(S) offset in the linear predictor
- Feature-level coefficients, p-values, and q-values from DE testing with depth accounted for

## How to apply

Compute a cell-specific (or sample-specific) library size or normalization factor S — for single-cell data the paper uses the SCRAN R/Bioconductor package to estimate cell-specific sizes. In the per-gene model, specify the linear predictor as log(mu) = X*beta + log(S), i.e., S enters as an offset on the log link so it is fixed rather than estimated. For metabolomics/microbiome inputs, compute the library size (or normalization factor) per sample the same way before the two-group design fit. Because the Tweedieverse framework uses the offset to account for sequencing depth, no pre-normalization or count transformation (e.g., CPM, log-transform) should be applied; the offset bypasses the need for an ad hoc normalization prior to DE testing.

## Related tools

- **SCRAN** (R/Bioconductor package used to estimate cell-specific library sizes that are incorporated as offsets in per-gene Tweedie models)
- **Tweedieverse** (Differential analysis framework that accepts the log library size as an offset in the log-link GLM (CPLM/ZICP)) — github.com/himelmallick/Tweedieverse
- **cplm (R package)** (Fits the Compound Poisson Linear Models whose linear predictor includes log(S) as an offset) — github.com/himelmallick/Tweedieverse

## Evaluation signals

- Check that S is strictly positive for every cell/sample before taking log(S); the offset must enter as log(S) on the link scale, not on the response scale
- Confirm S is supplied as a fixed offset (coefficient not estimated) and that the same S is reused across all per-gene fits for a given dataset
- Verify no prior transformation/normalization step (e.g., size-factor scaling of counts) was applied upstream, since the offset replaces it
- Compare estimated library sizes against sequencing depth/total counts per cell; SCRAN estimates should track total depth while correcting for compositional differences

## Limitations

- Requires a reliable depth measure; if cell-specific library sizes are poorly estimated, the offset propagates bias into all per-gene tests
- The paper's approach relies on this offset rather than normalization, so it presumes depth differences are the dominant technical confounder; platform-specific batch effects are not absorbed by the offset

## Evidence

- [readme] Nearly all these methods rely on some form of normalization (to account for differences in library size or sequencing depth): [evidence span withheld — anchor status: not_found]
- [readme] by utilizing the cellular sequencing depth as an offset, we bypass the need for an adhoc normalization and/or transformation prior to DE testing: [evidence span withheld — anchor status: not_found]
