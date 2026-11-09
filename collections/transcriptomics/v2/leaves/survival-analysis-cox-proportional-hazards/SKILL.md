---
name: survival-analysis-cox-proportional-hazards
description: Use when TCGA gene expression matrices and patient clinical metadata
  (vital status, days to death, days to follow-up) are available and gene sets (e.g.
  network-derived IC genes) must be tested for prognostic association with overall
  survival and compared via Concordance statistic.
when_to_use_negative:
- The dataset lacks patient follow-up/survival metadata (vital status and time-to-event),
  so Cox models cannot be fit — use the normal/cancer classification evaluation instead
- The goal is identifying biomarkers from raw co-expression networks rather than evaluating
  already-extracted IC gene lists against survival
- You need differential expression analysis itself; the Cox models here consume precomputed
  DE gene sets from DESeq2 as a baseline, not as the analysis target
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Python
  role: Language used to fit the univariate and multivariate Cox proportional-hazards
    models, extract hazard ratios and Concordance statistics, and write cox_survival_results.csv
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: Jupyter Notebook
  role: Notebook containing the systematic and statistic evaluation of the extracted
    IC genes, including the survival analysis
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb
- name: R/Bioconductor
  role: Hosts the DESeq2 package used to produce the DE gene baseline sets against
    which the Cox model Concordance statistics are compared
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:51b386e61a9caa4d65d93dd81756aa8f7afb5b5ba1650d7bc30d113399447b83
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# survival-analysis-cox-proportional-hazards

## Summary

Fits univariate and multivariate Cox proportional-hazards models (h(t) = h0(t) * exp(b*x)) to test whether IC genes extracted from fused co-expression networks associate with patient overall survival in TCGA KIRC, LIHC and PRAD, and compares Concordance statistics against a DESeq2-derived DE gene baseline.

## When to use

Use when TCGA gene expression matrices and patient clinical metadata (vital status, days to death, days to follow-up) are available and gene sets (e.g. network-derived IC genes) must be tested for prognostic association with overall survival and compared via Concordance statistic.

## When NOT to use

- The dataset lacks patient follow-up/survival metadata (vital status and time-to-event), so Cox models cannot be fit — use the normal/cancer classification evaluation instead
- The goal is identifying biomarkers from raw co-expression networks rather than evaluating already-extracted IC gene lists against survival
- You need differential expression analysis itself; the Cox models here consume precomputed DE gene sets from DESeq2 as a baseline, not as the analysis target

## Inputs

- TCGA gene expression matrices for KIRC, LIHC, and PRAD
- TCGA patient clinical metadata (vital status, days to death, days to follow-up)
- IC gene lists per cancer type (Extracted/IC_<tumor>.csv)
- DE gene sets from DESeq2 (adjusted p-value < 0.05, |log2 fold change| > 2.0)

## Outputs

- Hazard-ratio table (exp(b) and p-value per IC and DE gene)
- Counts of genes significantly associated with survival per gene set
- Concordance statistic comparison of multivariate IC vs DE gene models
- cox_survival_results.csv

## How to apply

Load the TCGA expression matrices and clinical metadata (vital status, days to death, days to follow-up) per cancer type, plus the IC gene lists (Extracted/IC_<tumor>.csv) and the DESeq2 DE gene baseline (adjusted p-value < 0.05, |log2 fold change| > 2.0). Fit a univariate Cox proportional-hazards model for every IC and DE gene, extracting the hazard ratio exp(b) and its p-value per gene. Count genes significantly associated with the survival event probability per set, expecting substantially more significant IC genes (44) than DE genes (6). For each cancer type, fit a multivariate Cox model on the significant IC genes and compute the Concordance statistic, comparing it against the DE gene models. Write the hazard-ratio table, significant-gene counts, and C-statistic comparison to cox_survival_results.csv, validating that C = 0.745 vs 0.666 for KIRC and C = 0.923 vs 0.797 for PRAD is reproduced.

## Related tools

- **Python** (Language used to fit the univariate and multivariate Cox proportional-hazards models, extract hazard ratios and Concordance statistics, and write cox_survival_results.csv) — https://github.com/DEIB-GECO/GeneNetFusion
- **Jupyter Notebook** (Notebook containing the systematic and statistic evaluation of the extracted IC genes, including the survival analysis) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb
- **R/Bioconductor** (Hosts the DESeq2 package used to produce the DE gene baseline sets against which the Cox model Concordance statistics are compared)

## Examples

```
pip install -r requirements.txt
```

## Evaluation signals

- Significant-gene row counts match the reported values: 44 IC genes vs 6 DE genes significantly associated with survival
- Concordance statistics match reference values: C = 0.745 vs 0.666 for KIRC and C = 0.923 vs 0.797 for PRAD, with IC gene models higher than DE models
- cox_survival_results.csv exists and contains the hazard-ratio table, significant-gene counts, and C-statistic comparison
- Hazard ratios are internally consistent as exp(b) for each gene's fitted Cox coefficient
- The multivariate Cox models are fit only on the genes found significant in the univariate per-gene screen

## Limitations

- No statistical software, packages, or versions are named for the survival analyses (Cox proportional-hazards model), so the exact library is not pinned down
- No thresholds or parameters are reported for declaring survival association significance
- The repository README documents full data/scripts only for the LIHC tumor analysis; for KIRC and PRAD only the IC gene symbols are provided in the Results folder, so KIRC/PRAD survival runs need expression and clinical data sourced separately

## Evidence

- [discussion] Multivariate Cox model gives better Concordance statistic: "the multivariate Cox proportional-hazards model of the significant IC genes for each cancer type gives a better Concordance statistic"
- [discussion] 44 IC vs 6 DE genes significant for survival: "44 of all IC genes are significantly associated with the survival event probability, a relevantly higher number than the 6 differentially expressed genes significantly associated with survival."
- [readme] Notebook for statistical evaluation of IC genes: "contains systematic and statistic evaluation of the extracted genes (here called IC genes)."
- [intro] DE analysis as baseline for comparison: "We also compare our results with the ones from differential gene expression analysis, which is the usual baseline method to identify biomarkers in biology"
