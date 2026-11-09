---
name: multivariate-cox-model-concordance-evaluation
description: Use when ranked gene sets (e.g., IC genes from fused co-expression networks)
  must be validated against patient survival endpoints, comparing their Cox hazard
  ratios and Concordance statistics to a DE-gene baseline.
when_to_use_negative:
- The goal is normal/cancer classification or network construction (use the repository's
  extraction_classification.py / Pipeline.ipynb workflow instead of survival modelling).
- No patient survival metadata (vital status, days to death/follow-up) is available
  for the cohort.
- Gene lists are not derived from fused co-expression networks or a comparable ranked
  signature and no DE baseline exists for fair comparison.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Implements the Cox model fitting, hazard-ratio extraction, and C-statistic
    comparison steps of the survival evaluation
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: Jupyter Notebook
  role: Hosts the evaluation notebooks (Pipeline.ipynb, Knowledge-based evaluation
    of the results.ipynb) that run the systematic/statistic evaluation of the extracted
    IC genes
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb
- name: R/Bioconductor
  role: Provides DESeq2 to generate the DE-gene baseline gene sets against which the
    IC-gene survival models are compared
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:50a61e3cc4a24c7ee781d972f1b4386327e9d6ac75dcb3e3a5f449aa81472138
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/GeneNetFusion
  tools:
  - Jupyter Notebook
  - Python
  - R/Bioconductor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# multivariate-cox-model-concordance-evaluation

## Summary

Prognostic evaluation of fused-network-derived IC genes against DESeq2-derived DE genes using univariate and multivariate Cox proportional-hazards models on TCGA KIRC, LIHC and PRAD patients, comparing Concordance statistics to show IC genes better fit patient overall survival.

## When to use

Use when ranked gene sets (e.g., IC genes from fused co-expression networks) must be validated against patient survival endpoints, comparing their Cox hazard ratios and Concordance statistics to a DE-gene baseline.

## When NOT to use

- The goal is normal/cancer classification or network construction (use the repository's extraction_classification.py / Pipeline.ipynb workflow instead of survival modelling).
- No patient survival metadata (vital status, days to death/follow-up) is available for the cohort.
- Gene lists are not derived from fused co-expression networks or a comparable ranked signature and no DE baseline exists for fair comparison.

## Inputs

- TCGA gene expression matrices for KIRC, LIHC, and PRAD (retrieved via GMQL/PyGMQL)
- TCGA patient clinical metadata: vital status, days to death, days to follow-up
- IC gene lists per cancer type from Extracted/IC_<tumor>.csv
- DE gene sets from DESeq2 (adjusted p-value < 0.05, |log2 fold change| > 2.0)

## Outputs

- Per-gene hazard-ratio table (exp(b) and p-value from univariate Cox models for IC and DE genes)
- Counts of genes significantly associated with the survival event probability (reported: 44 IC vs 6 DE)
- Concordance statistic comparison between multivariate IC-gene models and DE-gene models
- cox_survival_results.csv

## How to apply

Load TCGA gene expression matrices and clinical metadata (vital status, days to death, days to follow-up) for KIRC, LIHC, and PRAD, plus the IC gene lists (Extracted/IC_<tumor>.csv) and the DESeq2 baseline gene sets (adjusted p-value < 0.05, |log2 fold change| > 2.0). Fit a univariate Cox PH model per gene with h(t) = h0(t) * exp(b*x), extracting exp(b) hazard ratios and p-values for both IC and DE sets, then count significantly associated genes — expected 44 IC vs 6 DE. Fit a multivariate Cox PH model on the significant IC genes per cancer type and compute the Concordance statistic, comparing against the DE-gene models (KIRC C = 0.745 vs 0.666; PRAD C = 0.923 vs 0.797). Write hazard-ratio tables, significant-gene counts, and the C-statistic comparison to cox_survival_results.csv. Note the paper names no specific survival-analysis package or version, so any standard Cox implementation in Python/R must be validated against the reported counts and statistics.

## Related tools

- **Python** (Implements the Cox model fitting, hazard-ratio extraction, and C-statistic comparison steps of the survival evaluation) — https://github.com/DEIB-GECO/GeneNetFusion
- **Jupyter Notebook** (Hosts the evaluation notebooks (Pipeline.ipynb, Knowledge-based evaluation of the results.ipynb) that run the systematic/statistic evaluation of the extracted IC genes) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb
- **R/Bioconductor** (Provides DESeq2 to generate the DE-gene baseline gene sets against which the IC-gene survival models are compared)

## Evaluation signals

- Row counts of significant genes match the reported values: 44 IC genes vs 6 DE genes significantly associated with survival.
- Concordance statistics of multivariate IC-gene Cox models exceed the DE-gene baselines for KIRC (C = 0.745 vs 0.666) and PRAD (C = 0.923 vs 0.797).
- cox_survival_results.csv exists and contains the hazard-ratio table, significant-gene counts, and C-statistic comparison.
- Every gene has an exp(b) hazard ratio and p-value from the univariate Cox model h(t) = h0(t) * exp(b*x).
- DE baseline gene sets satisfy the DESeq2 thresholds adjusted p-value < 0.05 and |log2 fold change| > 2.0.

## Limitations

- No statistical software, packages, or versions are named for the survival analyses or the Cox proportional-hazards models, so reproducibility depends on re-implementation.
- No thresholds or parameters are reported for declaring survival association significance beyond the observed 44 vs 6 gene counts.
- The reported C-statistic advantage covers KIRC and PRAD; the repository's data, scripts and results are documented for LIHC only, requiring adaptation for other tumors.

## Evidence

- [discussion] the multivariate Cox proportional-hazards model of the significant IC genes for each cancer type gives a better Concordance statistic: "the multivariate Cox proportional-hazards model of the significant IC genes for each cancer type gives a better Concordance statistic"
- [discussion] 44 of all IC genes are significantly associated with the survival event probability, a relevantly higher number than the 6 differentially expressed genes significantly associated with survival.: "44 of all IC genes are significantly associated with the survival event probability, a relevantly higher number than the 6 differentially expressed genes"
- [readme] This repository has been created to present the work done on computational analysis of fused co-expression networks for the identification of candidate cancer gene biomarkers.: "This repository has been created to present the work done on computational analysis of fused co-expression networks for the identification of candidate cancer gene biomarkers."
- [readme] contains systematic and statistic evaluation of the extracted genes (here called IC genes): "contains systematic and statistic evaluation of the extracted genes (here called IC genes)"
- [readme] one folder [Extracted](https://github.com/DEIB-GECO/GeneNetFusion/tree/master/Extracted) that contains the data regarding all the genes extracted with the pipeline: "one folder [Extracted](https://github.com/DEIB-GECO/GeneNetFusion/tree/master/Extracted) that contains the data regarding all the genes extracted with the pipeline"
