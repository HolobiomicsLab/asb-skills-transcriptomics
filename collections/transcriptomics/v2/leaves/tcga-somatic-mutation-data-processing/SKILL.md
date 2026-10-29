---
name: tcga-somatic-mutation-data-processing
description: Use when the analysis needs TCGA somatic mutation data downloaded and
  screened to a survival-analysis-ready design matrix — e.g., genes mutated in ≥1%
  of patients with patient overall survival times and censoring indicators as the
  outcome.
when_to_use_negative:
- The data are already a low-dimensional feature table or fitted model output, so
  no TCGA download or frequency screening is needed
- The covariate set is at the magnitude of a hundred thousand variables and penalized
  regression falls short — FDR first-step screening must be applied rather than fitting
  penalized models directly
- The goal is visualization-only exploration of mutations without survival or binary
  outcome regression
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_2640
tools:
- name: R
  role: Custom R codes download whole sets of molecular variables from TCGA and perform
    all statistical analysis of the mutation matrix and survival outcomes
- name: HDMAC
  role: Web-based platform (also runnable offline in RStudio) that applies preliminary
    mutation-frequency screening, optional FDR first-step screening, penalized Cox
    regression, and 5-fold cross validation on the processed data
  repo: https://github.com/chung-R/HD-MAC
- name: cBioPortal
  role: Alternative source for TCGA data downloads, though only limited numbers of
    genetic entries may be downloaded each time
- name: Shiny
  role: R package used to build HDMAC's interactive web interface that exposes the
    TCGA data processing and regression workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:46d7b02621ed7b5096e239a363cd2059749e1036cb21f4585782858588d9e695
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chung-R/HD-MAC
  tools:
  - HDMAC
  - R
  - Shiny
  - cBioPortal
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# tcga-somatic-mutation-data-processing

## Summary

Retrieves and preprocesses TCGA somatic mutation matrices (e.g., TCGA ovarian HGSC: 670 genes with ≥1% mutation frequency, 316 patients) plus survival times and censoring indicators, so the high-dimensional covariates are ready for HDMAC's penalized Cox regressions (Ridge, Lasso, adaptive Lasso) with optional FDR first-step screening.

## When to use

Use when the analysis needs TCGA somatic mutation data downloaded and screened to a survival-analysis-ready design matrix — e.g., genes mutated in ≥1% of patients with patient overall survival times and censoring indicators as the outcome.

## When NOT to use

- The data are already a low-dimensional feature table or fitted model output, so no TCGA download or frequency screening is needed
- The covariate set is at the magnitude of a hundred thousand variables and penalized regression falls short — FDR first-step screening must be applied rather than fitting penalized models directly
- The goal is visualization-only exploration of mutations without survival or binary outcome regression

## Inputs

- TCGA somatic mutation matrix (binary gene x patient alteration calls), e.g., TCGA ovarian high-grade serous carcinoma
- Patient overall survival times with censoring indicators (316 patients for the ovarian HGSC example)
- Optional mRNA expression Z-scores (abnormal if Z > 2 or Z < −2)
- Optional clinical covariates (e.g., gender, age)

## Outputs

- Mutation-frequency-screened gene matrix (670 genes retained at the ≥1% threshold for ovarian HGSC)
- FDR-screened candidate gene list with hazard ratios (ZSWIM8 HR 7.493, PABPC3 HR 5.635)
- Summary table of model gene counts and 5-fold cross-validated C-indices (e.g., ovarian_mutation_cox_cindex_table.csv)

## How to apply

Download the TCGA molecular data with the custom R codes provided in the HDMAC GitHub repository (https://github.com/chung-R/HD-MAC), since cBioPortal only allows limited numbers of genetic entries per download. Apply the preliminary screening so that only genes whose mutations were found in 1% or more of the patients are retained — for the TCGA ovarian HGSC data this yields 670 mutated genes across 316 patients. If mRNA expression is also processed, flag abnormal expression with a Z score above 2 or below −2, retaining entries abnormal in 2% or more of patients. Pair the filtered mutation matrix with the patients' overall survival times and censoring indicators, and optionally apply Benjamini-Hochberg FDR control (default threshold 0.05) as a first-step screening before fitting the penalized Cox models; FDR screening is recommended when variables reach the magnitude of a hundred thousand. Evaluate the downstream models with 5-fold cross validation (the HDMAC default fold number) using the concordance index.

## Related tools

- **R** (Custom R codes download whole sets of molecular variables from TCGA and perform all statistical analysis of the mutation matrix and survival outcomes)
- **HDMAC** (Web-based platform (also runnable offline in RStudio) that applies preliminary mutation-frequency screening, optional FDR first-step screening, penalized Cox regression, and 5-fold cross validation on the processed data) — https://github.com/chung-R/HD-MAC
- **cBioPortal** (Alternative source for TCGA data downloads, though only limited numbers of genetic entries may be downloaded each time)
- **Shiny** (R package used to build HDMAC's interactive web interface that exposes the TCGA data processing and regression workflow)

## Examples

```
git clone https://github.com/chung-R/HD-MAC  # then run the provided TCGA-download R script in RStudio to obtain the somatic mutation and survival data
```

## Evaluation signals

- After preliminary screening, only genes with mutations found in ≥1% of patients are retained; for TCGA ovarian HGSC this must be 670 mutated genes across 316 patients
- If FDR screening is applied at the default 0.05 threshold, the selected gene list must contain exactly the FDR-passing candidates (ZSWIM8 and PABPC3 for the ovarian data)
- Cross validation must use the default fold number of 5 (no cross validation if 1 is chosen), yielding C-index values comparable to the reported ovarian table (0.529/0.501/0.501 without FDR; 0.502/0.502/0.497 with FDR)
- The processed dataset must carry both survival times and censoring indicators so that Cox (not logistic) regression is triggered
- Downloaded TCGA data should match the sample set that HDMAC's R scripts retrieve; cBioPortal-derived pulls may be incomplete due to per-download entry limits

## Limitations

- cBioPortal can download TCGA data but only limited numbers of genetic entries may be downloaded each time
- Penalized regression models fall short at variables of the magnitude of a hundred thousand; FDR first-step screening is recommended in that regime
- Genes identified from the processed data (e.g., ASAP3, ZSWIM8, PABPC3) are preliminary candidates and require further in-depth study
- Genome-wide search for genetic changes associated with ovarian cancer survival is still waiting

## Evidence

- [abstract] R codes are also provided to facilitate download of whole sets of molecular variables from TCGA: "R codes are also provided to facilitate download of whole sets of molecular variables from TCGA"
- [methods] We therefore wrote R codes to download large numbers of genomic data from TCGA: "We therefore wrote R codes to download large numbers of genomic data from TCGA"
- [methods] only the genes whose mutations were found in 1% or more and the mRNA entries whose abnormal expression was found in 2% or more of the patients were: "only the genes whose mutations were found in 1% or more and the mRNA entries whose abnormal expression was found in 2% or more of the patients were"
- [methods] It is also possible to download TCGA data from cBioPortal, but only limited numbers of genetic entries may be downloaded each time: "It is also possible to download TCGA data from cBioPortal, but only limited numbers of genetic entries may be downloaded each time"
- [discussion] Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short: "Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short"
