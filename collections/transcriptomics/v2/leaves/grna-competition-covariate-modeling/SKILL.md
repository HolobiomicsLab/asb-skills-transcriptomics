---
name: grna-competition-covariate-modeling
description: Use when analyzing high-MOI single-cell CRISPR screens (e.g. the 207,324-cell
  K562 CRISPRi CROP-seq screen with 13,186 gRNAs) where multiple gRNAs are assayed
  per cell and FPR of gRNA-gene associations must be controlled against NTC-based
  estimates.
when_to_use_negative:
- Datasets without multiple gRNAs per cell (low-MOI screens), where no gRNA cross-associations
  arise and untested-gRNA covariates are unnecessary
- Non-UMI full-length scRNA-seq platforms, since the authors state they 'only focused
  on UMI-based datasets' and nonlinear covariates 'may benefit from revalidating'
  on full-length platforms
- Hypothesis testing of arbitrary nonlinear associations, for which Normalisr is explicitly
  not designed
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: Normalisr
  role: Normalization (qc_reads, qc_outlier, lcpm, normcov, scaling_factor, compute_var,
    normvar) and linear association testing ('de' submodule) with untested gRNAs as
    fixed-effect covariates; installed via pip install normalisr
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:f9021ea2c060ba33ee3c2504c987f520bdf29c1388f61843b1edde672e15b5b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - Normalisr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# grna-competition-covariate-modeling

## Summary

Models gRNA cross-associations in high-MOI single-cell CRISPR screens by including untested gRNAs as fixed-effect covariates in the Normalisr linear association model (reduced to top-500 principal components when >10,000 untested gRNAs), preventing the false positive rate inflation seen with competition-naive differential expression.

## When to use

Use when analyzing high-MOI single-cell CRISPR screens (e.g. the 207,324-cell K562 CRISPRi CROP-seq screen with 13,186 gRNAs) where multiple gRNAs are assayed per cell and FPR of gRNA-gene associations must be controlled against NTC-based estimates.

## When NOT to use

- Datasets without multiple gRNAs per cell (low-MOI screens), where no gRNA cross-associations arise and untested-gRNA covariates are unnecessary
- Non-UMI full-length scRNA-seq platforms, since the authors state they 'only focused on UMI-based datasets' and nonlinear covariates 'may benefit from revalidating' on full-length platforms
- Hypothesis testing of arbitrary nonlinear associations, for which Normalisr is explicitly not designed

## Inputs

- UMI read count matrix (tsv or Cell Ranger sparse mtx, gzipped allowed) for the full-scale K562 CRISPRi screen (207,324 cells)
- gRNA assignment matrix for 13,186 gRNAs targeting 10,877 genes (from GEO deposition)
- NTC (non-targeting control) gRNA labels for FPR estimation

## Outputs

- gRNA-gene association table (de_associations.tsv) with log fold-changes, P values, and Q values for all 13,186 x 10,877 pairs from naive and competition-aware runs
- FPR estimates via Storey's method on NTC P-value histograms (e.g. ~1.5% competition-aware vs ~4.8% naive) and association-inflation ratio at Q<=0.2

## How to apply

Obtain the UMI read count matrix and gRNA assignment matrix from GEO, preprocess with Normalisr (qc_reads, qc_outlier, lcpm, normcov, scaling_factor, compute_var, normvar) to remove nonlinear library-size confounders at mean and variance levels. Run the Normalisr 'de' submodule in two configurations: a naive run without untested-gRNA covariates and a competition-aware run where all other gRNAs enter the linear model as fixed effects, reduced to their top-500 principal components when their number exceeds 10,000. Extract log fold-changes, P values, and Q values for every gRNA-gene pair from both runs. Estimate FPR by Storey's method on the P-value histograms of non-targeting-control (NTC) gRNAs, expecting ~4.8% naive vs ~1.5% competition-aware overall, and ~10.3% naive among TSS-targeted genes; also compute the association-inflation ratio at Q<=0.2. The rationale is that Normalisr's linear model supports 'native removal of covariates' such as untested gRNAs as fixed effects, so competition is modeled without permutation-based testing.

## Related tools

- **Normalisr** (Normalization (qc_reads, qc_outlier, lcpm, normcov, scaling_factor, compute_var, normvar) and linear association testing ('de' submodule) with untested gRNAs as fixed-effect covariates; installed via pip install normalisr) — https://github.com/lingfeiwang/normalisr

## Examples

```
normalisr de -h
```

## Evaluation signals

- Competition-aware run yields NTC-based overall FPR of ~1.5% (Storey's method) versus ~4.8% for the naive run
- FPR among NTC gRNAs for TSS-targeted genes is ~10.3% under the naive configuration, demonstrating detectable inflation
- Association-inflation ratio computed at Q<=0.2 should show fewer spurious gRNA-gene associations in the competition-aware run
- Output de_associations.tsv contains logFC, P, and Q columns for every gRNA-gene pair in both runs
- Number of untested-gRNA covariates reduced to top-500 principal components whenever their count exceeds 10,000

## Limitations

- SCEPTRE P values for non-cis-effects (including NTCs) were unavailable, preventing sensitivity/specificity cross-method comparison
- Only UMI-based datasets were analyzed; full-length platforms require revalidation of nonlinear cellular summary covariates
- Batch effects in multi-dataset integrative studies are not addressed by Normalisr

## Evidence

- [readme] native removal of covariates (e.g. batches, house-keeping programs, and untested gRNAs) as fixed effects: "native removal of covariates (*e.g.* batches, house-keeping programs, and untested gRNAs) as fixed effects"
- [discussion] UMI read count and gRNA assignment matrices were downloaded from Gene Expression Omnibus (GEO): "UMI read count and gRNA assignment matrices were downloaded from Gene Expression Omnibus (GEO)"
- [abstract] systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels: "By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels"
- [methods] QC and normalization submodule list from methods: "qc_reads
	qc_outlier"
- [full_text] a competition-naive DE analysis using Normalisr that disregarded other, untested gRNAs led to a 4.8% overall FPR among NTCs: [evidence span withheld — anchor status: not_found]
