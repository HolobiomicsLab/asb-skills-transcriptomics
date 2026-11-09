---
name: spearman-rank-correlation
description: Use when you have per-sample gene-signature scores and a continuous per-sample
  metadata variable (purity/VAF) in a pdacR dataset and need a rank-based association
  with rho, p-value, and n.
when_to_use_negative:
- The comparison variable is categorical or the relationship is expected to be linear
  with Gaussian residuals where parametric correlation would be preferred
- The dataset has not been filtered to confirmed PDAC samples (e.g., using all 182
  TCGA-PAAD samples instead of the 150 whitelisted ones), which the article warns
  can introduce non-specificity
- The goal is single-cell population separation, where bulk transcriptomic data cannot
  fully separate cellular populations
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: pdacR
  role: Provides parsed datasets ($ex, $sampInfo), curated gene signature lists, and
    sample-track filtering used to build the signature scores and purity variables
    before correlation
  repo: github.com/rmoffitt/pdacR
- name: R
  role: Runtime for cor.test(method = 'spearman') and the pdacR package
- name: ggplot2
  role: Renders the scatter plot of VAF versus exocrine score with fitted trend line
    for figure reproduction
- name: devtools
  role: Installs pdacR via devtools::install_github('rmoffitt/pdacR')
  repo: rmoffitt/pdacR
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:e1b44f3c5009a17cd74e9a6feaee888d7b6d08ed93bcdabdbafc72f560c42e74
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - devtools
  - ggplot2
  - pdacR
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# spearman-rank-correlation

## Summary

Compute nonparametric Spearman rank correlations between PDAC gene-signature scores (e.g., Collisson Exocrine, Bailey ADEX) and sample purity proxies (predicted mean VAF or ABSOLUTE purity) using cor.test(method = 'spearman') in R on pdacR-parsed datasets, to test whether a signature predicts tumor cellularity.

## When to use

Use when you have per-sample gene-signature scores and a continuous per-sample metadata variable (purity/VAF) in a pdacR dataset and need a rank-based association with rho, p-value, and n.

## When NOT to use

- The comparison variable is categorical or the relationship is expected to be linear with Gaussian residuals where parametric correlation would be preferred
- The dataset has not been filtered to confirmed PDAC samples (e.g., using all 182 TCGA-PAAD samples instead of the 150 whitelisted ones), which the article warns can introduce non-specificity
- The goal is single-cell population separation, where bulk transcriptomic data cannot fully separate cellular populations

## Inputs

- pdacR parsed expression matrix ($ex) with gene rows matched at SYMBOL level
- Sample-level metadata ($sampInfo) containing purity proxies (predicted mean VAF or ABSOLUTE purity)
- Curated gene signature lists (gene_lists) from pdacR's Data/ directory (Collisson Exocrine, Bailey ADEX)
- Sample-track filter specification (e.g., 'Decision' = 'whitelist' for TCGA-PAAD)

## Outputs

- Spearman rho estimate, p-value, and sample size n (e.g., rho = -0.260, p = 0.0014, n = 150)
- ggplot2 scatter plot of purity proxy versus signature score with fitted trend line reproducing Fig. 2a/2b

## How to apply

Load a parsed pdacR dataset (e.g., Puleo 2018 microarray via parse_Puleo_Array or TCGA-PAAD $ex/$sampInfo) plus the curated gene_lists object from the package Data/ directory. Match signature genes at the SYMBOL level, subset to the correct samples (e.g., the 309 FFPE primary PDAC samples in Puleo; the 150 'whitelist' histologically confirmed PDAC tumors in TCGA-PAAD via the 'Decision' filter), and score each sample as the mean log2 expression of the signature genes from $ex. Pull the purity proxy per sample from $sampInfo (predicted mean VAF for Puleo, ABSOLUTE-estimated purity for TCGA) and run cor.test(score, purity, method = 'spearman') in R. Render a ggplot2 scatter plot with a fitted trend line annotated with rho, p, and n, cross-checking against the paper's figures (Fig. 2a/2b, Supplementary Fig. 1A); expected benchmarks are rho = -0.260, p = 0.0014, n = 150 (ADEX vs ABSOLUTE) and rho = -0.221, p = 0.0003 (Exocrine vs VAF).

## Related tools

- **pdacR** (Provides parsed datasets ($ex, $sampInfo), curated gene signature lists, and sample-track filtering used to build the signature scores and purity variables before correlation) — github.com/rmoffitt/pdacR
- **R** (Runtime for cor.test(method = 'spearman') and the pdacR package)
- **ggplot2** (Renders the scatter plot of VAF versus exocrine score with fitted trend line for figure reproduction)
- **devtools** (Installs pdacR via devtools::install_github('rmoffitt/pdacR')) — rmoffitt/pdacR

## Examples

```
devtools::install_github('rmoffitt/pdacR'); ct <- cor.test(adex_score, purity, method = 'spearman')
```

## Evaluation signals

- TCGA-PAAD ADEX-vs-ABSOLUTE run reports exactly n = 150 whitelisted PDAC tumors
- Reported rho lies within [-1, 1] with an associated p-value, and rho/p/n match Fig. 2a values (rho = -0.260, p = 0.0014) or Puleo values (rho = -0.221, p = 0.0003)
- The rendered scatter plot reproduces the association shown in Fig. 2b / Supplementary Fig. 1A, including a negative trend line
- Signature scoring uses the correct gene matching level (SYMBOL) and the intended sample subset (309 FFPE primary PDAC for Puleo; 150 whitelist for TCGA)

## Limitations

- TCGA-PAAD contains 182 samples but only 150 are histologically confirmed PDAC; including the other 32 can mislead signature derivation or validation
- Bulk transcriptomic data limit the ability to fully separate cellular populations, so purity proxies (predicted VAF, ABSOLUTE) are indirect measures
- Datasets span multiple platforms (bulk RNA-seq and microarrays), so correlations are not directly comparable across datasets without curation

## Evidence

- [other] The Collisson Exocrine gene signature is negatively correlated with sample purity in the Puleo 2018 dataset (Spearman rho = −0.221, p = 0.0003): [evidence span withheld — anchor status: not_found]
- [other] In the TCGA dataset, high expression of the Bailey ADEX gene signature in PDAC tumors predicts low tumor cellularity as estimated by ABSOLUTE (Spearman rho = −0.260, p = 0.0014, n = 150).: [evidence span withheld — anchor status: not_found]
- [intro] The TCGA PDAC mRNA dataset contains 182 samples but only 150 are histologically confirmed PDAC; including the other 32 samples misleads signature derivation or validation: "The mRNA dataset includes 182 samples, but only 150 have been confirmed histologically as PDAC"
- [readme] Filter to the correct 150 whitelist samples via the Decision sample track: "To filter to the correct 150, select "Decision", and then remove all variables except "whitelist"."
- [methods] Installing pdacR: "devtools::install_github("rmoffitt/pdacR")"
