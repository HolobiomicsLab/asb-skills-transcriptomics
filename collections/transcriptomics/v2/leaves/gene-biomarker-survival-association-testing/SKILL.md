---
name: gene-biomarker-survival-association-testing
description: Use when TCGA-style gene expression matrices are paired with patient
  survival metadata (vital status, days to death, days to follow-up) and candidate
  gene sets (e.g., Extracted/IC_<tumor>.csv vs DE genes) must be screened for significant
  survival association and compared by Concordance statistic.
when_to_use_negative:
- The goal is normal-vs-cancer classification performance of gene signatures rather
  than survival-time modeling — that is a separate evaluation in this paper.
- Clinical metadata lack survival times and event indicators (no days to death or
  follow-up), so Cox models cannot be fit.
- 'IC gene lists do not yet exist: the fused co-expression pipeline (Pipeline.ipynb
  plus Gephi community extraction) must run first to produce Extracted/IC_<tumor>.csv.'
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: Python
  role: Implements the univariate and multivariate Cox model fits and writes cox_survival_results.csv
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: Jupyter Notebook
  role: Hosts the systematic and statistical evaluation of the extracted IC genes
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb
- name: DESeq2
  role: Produces the baseline DE gene sets (adjusted p-value < 0.05, |log2 fold change|
    > 2.0) used as the comparator in the Cox survival analysis
- name: R/Bioconductor
  role: Environment for running DESeq2 to derive the DE-gene baseline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:4139afbe6cecb492578346eaafc56c0f14d9ef0927b497f625a21110755f6851
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/GeneNetFusion
  tools:
  - DESeq2
  - Jupyter Notebook
  - Python
  - R/Bioconductor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gene-biomarker-survival-association-testing

## Summary

Tests whether IC genes extracted from fused TCGA co-expression networks are prognostically associated with patient overall survival by fitting univariate Cox proportional-hazards models h(t) = h0(t) * exp(b*x) per gene, then comparing multivariate Cox Concordance statistics of significant IC genes against the DESeq2 DE-gene baseline.

## When to use

Use when TCGA-style gene expression matrices are paired with patient survival metadata (vital status, days to death, days to follow-up) and candidate gene sets (e.g., Extracted/IC_<tumor>.csv vs DE genes) must be screened for significant survival association and compared by Concordance statistic.

## When NOT to use

- The goal is normal-vs-cancer classification performance of gene signatures rather than survival-time modeling — that is a separate evaluation in this paper.
- Clinical metadata lack survival times and event indicators (no days to death or follow-up), so Cox models cannot be fit.
- IC gene lists do not yet exist: the fused co-expression pipeline (Pipeline.ipynb plus Gephi community extraction) must run first to produce Extracted/IC_<tumor>.csv.

## Inputs

- TCGA gene expression matrices for KIRC, LIHC, and PRAD (from GMQL/PyGMQL retrieval of TCGA data)
- Patient clinical metadata: vital status, days to death, days to last follow-up
- IC gene lists per cancer type: Extracted/IC_<tumor>.csv
- Baseline DE gene sets from DESeq2 (adjusted p-value < 0.05, |log2 fold change| > 2.0)

## Outputs

- Per-gene hazard-ratio table (hazard ratio exp(b) and p-value for each IC and DE gene)
- Significant-gene counts per gene set (reference: 44 IC vs 6 DE significantly associated with survival)
- Concordance statistic comparison between multivariate IC-gene and DE-gene Cox models
- cox_survival_results.csv containing hazard ratios, counts, and C-statistic comparison

## How to apply

Load the TCGA gene expression matrices and clinical metadata (vital status, days to death, days to follow-up) for KIRC, LIHC, and PRAD, plus the IC gene lists from Extracted/IC_<tumor>.csv and the DESeq2 DE-gene baselines (adjusted p-value < 0.05, |log2 fold change| > 2.0). For every IC gene and DE gene, fit a univariate Cox proportional-hazards model h(t) = h0(t) * exp(b*x), extracting the hazard ratio exp(b) and its p-value per gene. Count genes significantly associated with the survival event probability in each gene set — the reference run yields 44 IC vs 6 DE significant genes — and verify your counts match before proceeding. For each cancer type, fit a multivariate Cox model on the significant IC genes, compute the Concordance statistic, and compare against the DE-gene models (expected: KIRC C = 0.745 vs 0.666; PRAD C = 0.923 vs 0.797, favoring IC genes). Write the hazard-ratio table, significant-gene counts, and C-statistic comparison to cox_survival_results.csv.

## Related tools

- **Python** (Implements the univariate and multivariate Cox model fits and writes cox_survival_results.csv) — https://github.com/DEIB-GECO/GeneNetFusion
- **Jupyter Notebook** (Hosts the systematic and statistical evaluation of the extracted IC genes) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb
- **DESeq2** (Produces the baseline DE gene sets (adjusted p-value < 0.05, |log2 fold change| > 2.0) used as the comparator in the Cox survival analysis)
- **R/Bioconductor** (Environment for running DESeq2 to derive the DE-gene baseline)

## Evaluation signals

- Significant-gene counts match the reported 44 IC genes vs 6 DE genes significantly associated with survival.
- Concordance statistics of multivariate significant-IC-gene models match reference values: KIRC C = 0.745 (vs DE 0.666) and PRAD C = 0.923 (vs DE 0.797).
- cox_survival_results.csv exists and contains the hazard-ratio table, significant-gene counts, and C-statistic comparison.
- Row counts of the significant-gene tables equal the reported counts for every cancer type.
- Each gene's hazard ratio equals exp(b) from its univariate Cox fit h(t) = h0(t) * exp(b*x), with HR > 1 indicating increased risk and HR < 1 protection.

## Limitations

- The article names no statistical software, packages, or versions for the survival analyses or the Cox proportional-hazards models, so implementations must be chosen by the practitioner.
- No thresholds or parameters are reported for declaring survival-association significance (only the counts 44 IC vs 6 DE are given).
- The repository hosts data, scripts, and results only for the LIHC tumor analysis; Results contains IC gene symbols for LIHC, KIRC, and PRAD, but full expression datasets for the other types must be retrieved separately (e.g., via GMQL/PyGMQL).

## Evidence

- [discussion] the multivariate Cox proportional-hazards model of the significant IC genes for each cancer type gives a better Concordance statistic: "the multivariate Cox proportional-hazards model of the significant IC genes for each cancer type gives a better Concordance statistic"
- [discussion] 44 IC vs 6 DE significant genes: "44 of all IC genes are significantly associated with the survival event probability, a relevantly higher number than the 6 differentially expressed genes significantly associated with survival."
- [readme] Extracted folder contents: "one folder [Extracted] that contains the data regarding all the genes extracted with the pipeline"
