---
name: tcga-clinical-metadata-integration
description: Use when TCGA expression data must be joined with per-patient survival
  metadata to test gene-level prognostic association (hazard ratios) and compare Concordance
  statistics between candidate gene sets such as IC genes and DE genes.
when_to_use_negative:
- When the goal is differential expression itself rather than survival association
  — DE genes here are only the baseline input from DESeq2 (adjusted p-value < 0.05,
  |log2FC| > 2.0)
- When no per-patient survival metadata (vital status, days to death/follow-up) is
  available for the TCGA samples
- When analyzing cancer types other than KIRC, LIHC, and PRAD, for which no IC gene
  lists are provided in the repository
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: 'Implements the Cox survival workflow: joining expression matrices with clinical
    metadata, fitting Cox models, and writing cox_survival_results.csv'
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: Jupyter Notebook
  role: Hosts the pipeline and evaluation notebooks used to produce the IC gene sets
    and evaluate extracted genes
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: DESeq2
  role: R/Bioconductor package used to derive the baseline differentially expressed
    gene sets that are compared against IC genes in the survival models
- name: PyGMQL
  role: Retrieves the TCGA gene expression datasets used as input to the analysis
  repo: https://pygmql.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:0fc38a25abfea415659d01a6b6e6245ed0649c5ec0102a69f0caa0756e05bc99
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/GeneNetFusion
  tools:
  - DESeq2
  - Jupyter Notebook
  - PyGMQL
  - Python
  - R/Bioconductor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# tcga-clinical-metadata-integration

## Summary

Integrates TCGA gene expression matrices with patient clinical metadata (vital status, days to death, days to follow-up) to test, via Cox proportional-hazards models, whether fused-network IC genes are more strongly prognostic than DESeq2-derived differentially expressed genes across KIRC, LIHC, and PRAD.

## When to use

Use when TCGA expression data must be joined with per-patient survival metadata to test gene-level prognostic association (hazard ratios) and compare Concordance statistics between candidate gene sets such as IC genes and DE genes.

## When NOT to use

- When the goal is differential expression itself rather than survival association — DE genes here are only the baseline input from DESeq2 (adjusted p-value < 0.05, |log2FC| > 2.0)
- When no per-patient survival metadata (vital status, days to death/follow-up) is available for the TCGA samples
- When analyzing cancer types other than KIRC, LIHC, and PRAD, for which no IC gene lists are provided in the repository

## Inputs

- TCGA gene expression matrices for KIRC, LIHC, and PRAD
- Patient clinical metadata (vital status, days to death, days to follow-up)
- IC gene lists per cancer type (Extracted/IC_<tumor>.csv)
- DE gene sets obtained with DESeq2 (adjusted p-value < 0.05, |log2 fold change| > 2.0)

## Outputs

- Per-gene hazard ratio (exp(b)) and p-value table from univariate Cox models
- Count of genes significantly associated with the survival event probability per gene set
- Concordance statistics comparing multivariate IC-gene models vs DE-gene models
- cox_survival_results.csv

## How to apply

Load the TCGA gene expression matrices and patient clinical metadata (vital status, days to death, days to follow-up) for KIRC, LIHC, and PRAD, together with the IC gene lists from Extracted/IC_<tumor>.csv and the baseline DE gene sets identified with DESeq2 (adjusted p-value < 0.05, |log2 fold change| > 2.0). Fit a univariate Cox proportional-hazards model h(t) = h0(t) * exp(b*x) for each IC gene and each DE gene, extracting the hazard ratio exp(b) and its p-value per gene. Count genes significantly associated with the survival event probability in each gene set, verifying that IC gene sets yield substantially more significant genes than DE gene sets (reported: 44 IC vs 6 DE). Then fit a multivariate Cox model on the significant IC genes per cancer type and compute the Concordance statistic, comparing against the DE baseline (KIRC: C = 0.745 vs 0.666; PRAD: C = 0.923 vs 0.797). Write the hazard-ratio table, significant-gene counts, and C-statistic comparison to cox_survival_results.csv.

## Related tools

- **Python** (Implements the Cox survival workflow: joining expression matrices with clinical metadata, fitting Cox models, and writing cox_survival_results.csv) — https://github.com/DEIB-GECO/GeneNetFusion
- **Jupyter Notebook** (Hosts the pipeline and evaluation notebooks used to produce the IC gene sets and evaluate extracted genes) — https://github.com/DEIB-GECO/GeneNetFusion
- **DESeq2** (R/Bioconductor package used to derive the baseline differentially expressed gene sets that are compared against IC genes in the survival models)
- **PyGMQL** (Retrieves the TCGA gene expression datasets used as input to the analysis) — https://pygmql.readthedocs.io/en/latest/

## Examples

```
jupyter notebook "Knowledge-based evaluation of the results.ipynb"
```

## Evaluation signals

- Row counts of significant genes match the reported 44 IC genes vs 6 DE genes significantly associated with survival
- Concordance statistics match reference values: KIRC C = 0.745 vs 0.666 and PRAD C = 0.923 vs 0.797 (IC vs DE)
- cox_survival_results.csv exists and contains the hazard-ratio table, significant-gene counts, and C-statistic comparison
- Each reported hazard ratio equals exp(b) from the fitted Cox model h(t) = h0(t) * exp(b*x)
- DE gene baselines satisfy the DESeq2 thresholds (adjusted p-value < 0.05, |log2 fold change| > 2.0)

## Limitations

- No statistical software, packages, or versions are named in the paper for the survival analyses (Cox proportional-hazards model), so the implementation stack must be chosen by the practitioner
- No thresholds or parameters are reported for declaring survival association significance, so the per-gene significance cutoff must be selected explicitly
- Only KIRC, LIHC, and PRAD are covered; the repository provides IC gene results for these three cancer types only

## Evidence

- [discussion] Cox model performance: "the multivariate Cox proportional-hazards model of the significant IC genes for each cancer type gives a better Concordance statistic"
- [discussion] IC vs DE survival counts: "44 of all IC genes are significantly associated with the survival event probability, a relevantly higher number than the 6 differentially expressed genes significantly associated with survival."
- [methods] DE gene derivation: "We leverage the DESeq2 R/Bioconductor package to identify the differentially expressed (DE) genes"
- [readme] IC gene file convention: "Use [Gephi](https://gephi.org) in order to extract the relevant communities and save the genes in the folder [Extracted](https://github.com/DEIB-GECO/GeneNetFusion/tree/master/Extracted) with the"
- [readme] Evaluation notebook role: "The jupyter notebook [Knowledge-based evaluation of the results.ipynb](https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb) contains"
