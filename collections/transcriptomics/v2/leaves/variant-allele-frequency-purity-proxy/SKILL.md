---
name: variant-allele-frequency-purity-proxy
description: Use when analyzing bulk PDAC transcriptomic samples (e.g., the Puleo
  2018 FFPE primary cohort in pdacR) and you need a per-sample tumor purity proxy
  without matched normal sequencing, to validate that a gene signature tracks tumor
  content.
when_to_use_negative:
- Data are single-cell RNA-seq — cell populations can be resolved directly (see rmoffitt/scOh
  atlas), so a VAF purity proxy is unnecessary
- Your goal is subtyping (basal-like/classical or PurIST classification) rather than
  testing a signature's relationship to sample purity
- No sample-level VAF or other purity proxy metadata is available for your cohort,
  since the correlation cannot be computed without it
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: pdacR
  role: R package providing the parsed Puleo 2018 microarray dataset (parse_Pule_Array,
    $ex matrix, $sampInfo metadata) and curated gene signature lists used to score
    the Collisson Exocrine signature
  repo: https://github.com/rmoffitt/pdacR.git
- name: R
  role: Language in which the signature scoring and cor.test(method="spearman") correlation
    are executed
- name: ggplot2
  role: Used to render the VAF-vs-exocrine-score scatter plot with a fitted trend
    line reproducing Fig. 2b
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:43d5ab4043fcf720ff92b13dec79b62aea3f492304b34dce8ea35624c4e2cd66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rmoffitt/pdacR.git
  tools:
  - R
  - ggplot2
  - pdacR
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# variant-allele-frequency-purity-proxy

## Summary

Uses predicted mean variant allele frequency (VAF) per sample as a tumor-cellularity proxy for bulk PDAC microarray data, then tests whether curated gene signatures (e.g., Collisson Exocrine) correlate with purity via Spearman rank correlation.

## When to use

Use when analyzing bulk PDAC transcriptomic samples (e.g., the Puleo 2018 FFPE primary cohort in pdacR) and you need a per-sample tumor purity proxy without matched normal sequencing, to validate that a gene signature tracks tumor content.

## When NOT to use

- Data are single-cell RNA-seq — cell populations can be resolved directly (see rmoffitt/scOh atlas), so a VAF purity proxy is unnecessary
- Your goal is subtyping (basal-like/classical or PurIST classification) rather than testing a signature's relationship to sample purity
- No sample-level VAF or other purity proxy metadata is available for your cohort, since the correlation cannot be computed without it

## Inputs

- pdacR-parsed Puleo 2018 microarray dataset object (expression matrix in $ex, sample metadata in $sampInfo including predicted mean VAF)
- Curated gene signature lists (gene_lists) shipped in pdacR's Data/ directory (e.g., Collisson Exocrine symbols)

## Outputs

- Spearman correlation estimate and p-value (expected rho = -0.221, p = 0.0003 for Collisson Exocrine vs. VAF)
- Scatter plot of predicted mean VAF vs. exocrine signature score with fitted trend line (reproducing Fig. 2b / Supplementary Fig. 1A)

## How to apply

Load the parsed Puleo 2018 microarray dataset via pdacR's parse_Puleo_Array and the curated gene_lists from the package's Data/ directory. Extract the Collisson Exocrine gene symbols (SYMBOL-level matching) and subset the $ex expression matrix to the 309 FFPE primary PDAC samples. Score each sample's exocrine signature as the mean expression of its genes from $ex. Pull predicted mean VAF per sample from $sampInfo as the purity proxy, then run cor.test(method = "spearman") between signature score and VAF. Render a VAF-vs-score scatter plot with a fitted trend line (ggplot2) reproducing Fig. 2b, cross-checking against Supplementary Fig. 1A; rationale is that bulk data cannot fully separate cellular populations, so a molecular purity proxy (VAF) is used to test whether stromal/exocrine signatures are anti-correlated with tumor purity.

## Related tools

- **pdacR** (R package providing the parsed Puleo 2018 microarray dataset (parse_Pule_Array, $ex matrix, $sampInfo metadata) and curated gene signature lists used to score the Collisson Exocrine signature) — https://github.com/rmoffitt/pdacR.git
- **R** (Language in which the signature scoring and cor.test(method="spearman") correlation are executed)
- **ggplot2** (Used to render the VAF-vs-exocrine-score scatter plot with a fitted trend line reproducing Fig. 2b)

## Examples

```
cor.test(colMeans(ex[collisson_exocrine_syms, ]), sampInfo$predicted_mean_VAF, method = "spearman")
```

## Evaluation signals

- Spearman rho between Collisson Exocrine score and predicted mean VAF should be negative (reported rho = -0.221, p = 0.0003)
- Scatter plot of VAF vs. exocrine score with fitted trend line visually reproduces Fig. 2b and Supplementary Fig. 1A
- Exactly 309 FFPE primary PDAC samples are retained after subsetting the Puleo 2018 expression matrix
- Exocrine signature score equals the mean expression of the Collisson Exocrine gene symbols matched at the SYMBOL level, with no missing genes silently dropped

## Limitations

- Datasets are bulk analyses, limiting the ability to fully separate cellular populations — the VAF proxy is indirect and cannot resolve individual cell types
- Predicted mean VAF is a model-derived proxy, not measured tumor purity, so correlations reflect prediction error as well as biology
- Public PDAC data can contain low-purity and non-adenocarcinoma cases that introduce non-specificity; sample filtering must be applied before analysis

## Evidence

- [methods] workflow card, Puleo 2018 correlation: [evidence span withheld — anchor status: not_found]
- [methods] workflow card, VAF proxy: [evidence span withheld — anchor status: not_found]
- [discussion] bulk-data limitation: "Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations"
- [readme] gene signature usage in pdacR: "These gene sets can be used heatmapping, visualizing correlations, performing dimensionality reduction, and coloring differential expression plots."
- [readme] data curation location: "Data/ holds all parsed datasets, as well as the gene signatures object and PurIST classifier object."
