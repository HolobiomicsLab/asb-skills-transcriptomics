---
name: gene-wise-overdispersion-estimation-qc
description: Use when NEBULA (NEBULA-LN or NEBULA-HL) has been run gene-wise on multi-subject
  scRNA-seq/snRNA-seq counts and per-gene summary statistics include convergence codes
  plus subject-level and cell-level overdispersions that must be screened before downstream
  interpretation.
when_to_use_negative:
- The goal is differential expression on bulk RNA-seq or small-sample data designed
  for conventional transcriptomics tools rather than large-scale multi-subject single-cell
  data with a hierarchical overdispersion structure
- Only a handful of subjects are available and the variable of interest is subject-level
  — the paper notes testing subject-level variables is highly sensitive to the subject-level
  variance component and random-effects distribution assumptions when the number of
  subjects is small
- The input is already a filtered feature table with no per-gene overdispersion or
  convergence output, so NEBULA's QC criteria cannot be evaluated
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: nebula (R package v1.5.8)
  role: Fits the negative binomial (gamma) mixed model per gene via NEBULA-LN or NEBULA-HL
    and returns the overdispersion components and convergence codes that this skill
    screens
  repo: https://github.com/lhe17/nebula
- name: R (R-4.4.0)
  role: Runtime required by nebula v1.5.7+; the QC and filtering are performed on
    the R object returned by nebula()
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-021-02146-6
    title: NEBULA is a fast negative binomial mixed model for differential or co-expression
      analysis of large-scale multi-subject single-cell data
  source_content_hash: sha256:62e2f3781feaedf30c7d471a9d15a60e26eadcb1cabc2945a10238a7d1fa593f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lhe17/nebula
  tools:
  - NEBULA
  - NEBULA-LN
  - R (R-4.4.0)
  - Rcpp
  - Rfast
  - devtools
  - nebula (R package v1.5.8)
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-wise-overdispersion-estimation-qc

## Summary

Quality-controls per-gene overdispersion estimates (subject-level σ² and cell-level φ⁻¹) returned by NEBULA's negative binomial mixed model fits to multi-subject single-cell count data, removing genes whose overdispersion is too large or whose fit did not converge before interpreting fold changes and p-values.

## When to use

Use when NEBULA (NEBULA-LN or NEBULA-HL) has been run gene-wise on multi-subject scRNA-seq/snRNA-seq counts and per-gene summary statistics include convergence codes plus subject-level and cell-level overdispersions that must be screened before downstream interpretation.

## When NOT to use

- The goal is differential expression on bulk RNA-seq or small-sample data designed for conventional transcriptomics tools rather than large-scale multi-subject single-cell data with a hierarchical overdispersion structure
- Only a handful of subjects are available and the variable of interest is subject-level — the paper notes testing subject-level variables is highly sensitive to the subject-level variance component and random-effects distribution assumptions when the number of subjects is small
- The input is already a filtered feature table with no per-gene overdispersion or convergence output, so NEBULA's QC criteria cannot be evaluated

## Inputs

- NEBULA result object/list from nebula() containing per-gene summary statistics (logFC, se, raw p-values, subject-level overdispersion σ², cell-level overdispersion φ⁻¹, convergence codes)
- Raw integer count matrix (genes x cells; matrix or sparse dgCMatrix) with subject IDs used as nebula() input
- Design matrix from model.matrix with intercept term and unique column names

## Outputs

- Filtered per-gene summary table (e.g. nebula_rosmap_results.csv) with log fold change, raw p-values, subject-level and cell-level overdispersions, and convergence codes for retained genes
- Set of flagged genes: non-converged (code ≤ -20), cell-level overdispersion >100, and subject-level overdispersion >1 for subject-level predictors

## How to apply

After running nebula (which returns a list including logFC, raw p-values, subject-level overdispersion σ², cell-level overdispersion φ⁻¹, and a convergence code per gene), apply three QC screens grounded in the paper: (1) flag any gene whose convergence code is ≤ -20, since this signals the estimation algorithm did not converge and its summary statistics should be interpreted with caution or dropped; (2) for datasets with at least 50 cells per subject, remove genes with estimated cell-level overdispersion >100; (3) when the variable of interest is a subject-level predictor (e.g. disease status), remove or interpret cautiously genes with subject-level overdispersion >1, because testing subject-level variables is highly sensitive to estimation of the subject-level variance component. These thresholds can be combined with the low-expression pre-filter cpc=0.005 (counts per cell <0.5%) applied inside nebula. Note that NEBULA-LN trades some asymptotic efficiency in estimating cell-level overdispersion, and NEBULA-HL shows a downward bias in subject-level overdispersion, so overdispersion columns are QC quantities as well as biological signals (e.g. subject-level overdispersion may reflect cis-eQTL effects).

## Related tools

- **nebula (R package v1.5.8)** (Fits the negative binomial (gamma) mixed model per gene via NEBULA-LN or NEBULA-HL and returns the overdispersion components and convergence codes that this skill screens) — https://github.com/lhe17/nebula
- **R (R-4.4.0)** (Runtime required by nebula v1.5.7+; the QC and filtering are performed on the R object returned by nebula())

## Examples

```
re = nebula(sample_data$count, sample_data$sid, pred = df, ncore = 1); qc <- re$summary[re$summary$convergence > -20 & re$summary$cell_od < 100, ]
```

## Evaluation signals

- Every tested gene has a convergence code recorded; genes with code ≤ -20 are flagged/removed rather than silently kept
- After filtering, no retained gene has estimated cell-level overdispersion >100 (assuming ≥50 cells per subject in the input)
- For subject-level predictors, no retained gene has subject-level overdispersion >1 (or such genes are explicitly marked for cautious interpretation)
- The cpc=0.005 low-expression filter is reported in the nebula console output (number of genes removed having low expression) and matches the count of tested genes in the output table
- Output CSV contains all expected columns per gene: logFC, raw p-values, subject-level overdispersion, cell-level overdispersion, and convergence codes

## Limitations

- NEBULA-LN sacrifices some asymptotic efficiency in estimating the cell-level overdispersion compared with a standard h-likelihood method
- NEBULA-HL shows a downward bias in estimating the subject-level overdispersion, in line with theoretical results
- Testing a subject-level predictor is highly sensitive to the subject-level variance component and to random-effects distribution assumptions, and misspecification of the random-effects distribution may inflate false positives when the number of subjects is small
- Convergence code ≤ -20 means the algorithm did not converge well; results for such genes should be interpreted with caution

## Evidence

- [readme] The package *nebula* accounts for the hierarchical structure of the data by decomposing the total overdispersion into between-subject and within-subject components: "The package *nebula* accounts for the hierarchical structure of the data by decomposing the total overdispersion into between-subject and within-subject components"
- [readme] Generally, a convergence code $≤$ -20 suggests that the algorithm does not converge well.: "Generally, a convergence code $≤$ -20 suggests that the algorithm does not converge well. The results should be interpreted with caution in these cases."
- [methods] genes with an estimated cell-level overdispersion >100 should be removed for a data set with at least 50 cells per subject.: "genes with an estimated cell-level overdispersion >100 should be removed for a data set with at least 50 cells per subject."
- [methods] If the variable of interest is subject-level, genes with a very large subject-level overdispersion (>1) should be removed or interpreted cautiously as well.: "If the variable of interest is subject-level, genes with a very large subject-level overdispersion (>1) should be removed or interpreted cautiously as well."
- [discussion] Compared to a standard h-likelihood method, we see that NEBULA-LN sacrifices some asymptotic efficiency in estimating the cell-level overdispersion: "Compared to a standard h-likelihood method, we see that NEBULA-LN sacrifices some asymptotic efficiency in estimating the cell-level overdispersion"
