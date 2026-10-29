---
name: r-package-example-data-execution
description: Use when a newly installed R package ships with bundled example data
  (e.g. example.count, example.dat) and you need to verify the toolchain works end-to-end
  before applying it to real RNA-seq count data.
when_to_use_negative:
- The goal is statistical inference on your own dataset — this skill only validates
  the toolchain on bundled toy data and should not replace a real analysis run
- The package under test has no bundled example data or documented vignette, leaving
  no ground truth to validate against
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: kimma
  role: Target R package whose kmFit entry point and bundled example datasets (example.count,
    example.dat) are executed
  repo: https://github.com/BIGslu/kimma
- name: R
  role: Runtime environment in which the package is installed and the example-data
    execution is run
- name: devtools
  role: Used to install the package from GitHub (devtools::install_github("BIGslu/kimma"))
    before execution
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:4f9e38c559b805eb2d56aecebfd469bc6c1863d45d68519caeb535471931f791
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIGslu/kimma
  tools:
  - R
  - devtools
  - kimma
  - stats
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# r-package-example-data-execution

## Summary

Execute an R package's core functions against its bundled example datasets to validate installation and reproduce the documented workflow, as demonstrated by running kimma's kmFit on example.count and example.dat.

## When to use

Use when a newly installed R package ships with bundled example data (e.g. example.count, example.dat) and you need to verify the toolchain works end-to-end before applying it to real RNA-seq count data.

## When NOT to use

- The goal is statistical inference on your own dataset — this skill only validates the toolchain on bundled toy data and should not replace a real analysis run
- The package under test has no bundled example data or documented vignette, leaving no ground truth to validate against

## Inputs

- Bundled R package example datasets (e.g. kimma's example.count gene counts and example.dat sample metadata)
- Optionally example.kin (kinship covariance) and example.voom (precision weights) for richer model modes
- Package installation via devtools from https://github.com/BIGslu/kimma

## Outputs

- Per-gene results table with coefficient estimates, fit metrics (e.g. AIC), p-values and FDR (e.g. written to kimma_lm_results.csv)
- Console confirmation that kmFit completed without errors on the example data

## How to apply

Install the package from its GitHub source with devtools::install_github (e.g. BIGslu/kimma), then attach the bundled example objects — example.count (gene counts), example.dat (sample metadata), and, where needed, example.kin (kinship matrix) or example.voom (voom weights). Call the package's main entry point (kmFit) on these objects with the documented mode flags, e.g. run_lm=TRUE for the unpaired linear model, supplying counts, meta, and genes so expression is handled as TMM-normalized log2 CPM. Cross-check the returned results table (coefficient estimates, fit metrics such as AIC, p-values/FDR) against the package's instructional vignette at bigslu.github.io. Rationale: exercising the vignette's exact example data confirms the install, function exports (kimma_lm, kimma_lme, summarise_kmFit, etc.), and output schema are all functional before real-data use.

## Related tools

- **kimma** (Target R package whose kmFit entry point and bundled example datasets (example.count, example.dat) are executed) — https://github.com/BIGslu/kimma
- **R** (Runtime environment in which the package is installed and the example-data execution is run)
- **devtools** (Used to install the package from GitHub (devtools::install_github("BIGslu/kimma")) before execution)

## Examples

```
install.packages("devtools"); devtools::install_github("BIGslu/kimma"); library(kimma); kmFit(dat = example.count, meta = example.dat, genes = NULL, run_lm = TRUE, metrics = TRUE)
```

## Evaluation signals

- kmFit runs to completion on example.count/example.dat without errors after devtools::install_github
- Output table contains one record per modeled gene with estimate, fit-metric, and p-value/FDR fields
- Results are consistent with the lm-mode run shown in the kimma vignette at bigslu.github.io/kimma_vignette
- Written CSV (e.g. kimma_lm_results.csv) exists and is non-empty
- Function exports listed in the NAMESPACE (kimma_lm, kimma_lme, summarise_kmFit, kmFit_contrast) are callable after attach

## Limitations

- Example datasets (example.count, example.dat, example.kin) are small demonstration objects; successful execution does not certify behavior on full-scale RNA-seq data
- Unpaired lm mode ignores weights, random effects, and kinship covariance — exercising other modes (kimma_lme, kimma_lmerel) requires example.kin or example.voom

## Evidence

- [intro] installation from intro: "install.packages("devtools")
devtools::install_github("BIGslu/kimma")"
- [methods] example datasets in methods: "kmFit(dat, kin, patientID, libraryID, counts, meta, genes, weights, subset_var, subset_lvl"
- [methods] kimma_lm filter step: "kimma_lm(model_lm, to_model_gene, gene, use_weights, metrics)"
- [readme] README vignette pointer: "Kimma is freely available on GitHub https://github.com/BIGslu/kimma with an instructional vignette at https://bigslu.github.io/kimma_vignette/kimma_vignette.html"
- [abstract] package scope from abstract: "an open-source R package for flexible linear mixed effects modeling including covariates, weights, random effects, covariance matrices, and fit metrics"
