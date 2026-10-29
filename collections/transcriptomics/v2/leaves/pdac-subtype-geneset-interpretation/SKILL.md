---
name: pdac-subtype-geneset-interpretation
description: Use when a PDAC transcriptomic analysis needs to check whether a subtype
  gene signature (Collisson Exocrine, Bailey ADEX, etc.) actually tracks tumor cellularity
  rather than tumor-intrinsic biology, using pdacR datasets with purity or VAF metadata.
when_to_use_negative:
- The analysis goal is tumor-intrinsic subtype classification (e.g. PurIST) rather
  than testing whether a signature tracks sample purity/cellularity
- The dataset is single-cell RNA-seq (use the rmoffitt/scOh atlas instead); the article
  notes bulk datasets 'are limited in our ability to fully separate cellular populations'
- Samples have not been filtered to histologically confirmed PDAC (e.g. the 32 non-PDAC
  TCGA-PAAD samples), which would confound the signature-purity association
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: pdacR
  role: Provides the parsed PDAC datasets ($ex, $sampInfo), curated gene_lists signatures,
    and sample-track filters used to score signatures and correlate with purity
  repo: github.com/rmoffitt/pdacR
- name: R
  role: Analysis environment; cor.test(method='spearman') computes the signature-purity
    correlation
- name: ggplot2
  role: Renders the scatter plot of purity proxy versus signature score with a fitted
    trend line
- name: devtools
  role: Installs the pdacR package and its bundled datasets from GitHub
  repo: rmoffitt/pdacR
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:f0be3413b76d427a101bc767dc7af3c86474b097290f4a8343ed4092667b4627
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - devtools
  - ggplot2
  - pdacR
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# pdac-subtype-geneset-interpretation

## Summary

Scores a curated PDAC subtype gene signature (e.g. Collisson Exocrine or Bailey ADEX) per sample in pdacR datasets and tests its Spearman correlation with a tumor purity/cellularity proxy (ABSOLUTE purity or predicted mean VAF), establishing whether the signature marks low-purity, non-tumor-heavy bulk samples.

## When to use

Use when a PDAC transcriptomic analysis needs to check whether a subtype gene signature (Collisson Exocrine, Bailey ADEX, etc.) actually tracks tumor cellularity rather than tumor-intrinsic biology, using pdacR datasets with purity or VAF metadata.

## When NOT to use

- The analysis goal is tumor-intrinsic subtype classification (e.g. PurIST) rather than testing whether a signature tracks sample purity/cellularity
- The dataset is single-cell RNA-seq (use the rmoffitt/scOh atlas instead); the article notes bulk datasets 'are limited in our ability to fully separate cellular populations'
- Samples have not been filtered to histologically confirmed PDAC (e.g. the 32 non-PDAC TCGA-PAAD samples), which would confound the signature-purity association

## Inputs

- pdacR parsed dataset: $ex expression matrix (genes x samples) and $sampInfo sample-level metadata (including ABSOLUTE purity or predicted mean VAF)
- pdacR gene_lists object of curated PDAC subtype gene signatures (matched at SYMBOL level)
- Sample-track filter labels (e.g. Decision/whitelist for TCGA-PAAD)

## Outputs

- Spearman rho, p-value, and n for the signature-vs-purity correlation
- ggplot2 scatter plot of purity proxy versus signature score with fitted trend line (reproducing Fig. 2a / Fig. 2b / Supplementary Fig. 1A)
- Saved correlation estimate and figure

## How to apply

Install pdacR from github.com/rmoffitt/pdacR via devtools::install_github and load the parsed dataset's $ex expression matrix and $sampInfo sample metadata plus the gene_lists object from the package's Data/ directory. Curate the sample set before scoring: for TCGA-PAAD apply the 'Decision' filter keeping only 'whitelist' samples so the 182-sample dataset is reduced to the 150 histologically confirmed PDACs; for the Puleo 2018 microarray dataset restrict to the 309 FFPE primary PDAC samples. Match signature genes at the SYMBOL level, score each sample as the mean expression of the signature's genes, then run cor.test(method = 'spearman') between the signature score and the purity proxy (ABSOLUTE purity for TCGA; predicted mean VAF for Puleo). A negative rho (e.g. ADEX rho = -0.260, p = 0.0014, n = 150; Exocrine rho = -0.221, p = 0.0003) validates that the signature predicts low tumor purity, and the association is rendered as a ggplot2 scatter plot reproducing Fig. 2a/2b.

## Related tools

- **pdacR** (Provides the parsed PDAC datasets ($ex, $sampInfo), curated gene_lists signatures, and sample-track filters used to score signatures and correlate with purity) — github.com/rmoffitt/pdacR
- **R** (Analysis environment; cor.test(method='spearman') computes the signature-purity correlation)
- **ggplot2** (Renders the scatter plot of purity proxy versus signature score with a fitted trend line)
- **devtools** (Installs the pdacR package and its bundled datasets from GitHub) — rmoffitt/pdacR

## Examples

```
devtools::install_github("rmoffitt/pdacR"); cor.test(exocrine_score, vaf_proxy, method = "spearman")
```

## Evaluation signals

- TCGA run reports exactly n = 150 whitelisted PDAC tumors after the Decision/whitelist filter (not 182)
- Spearman rho is negative and within [-1, 1] with an associated p-value (e.g. ADEX rho = -0.260, p = 0.0014; Collisson Exocrine rho = -0.221, p = 0.0003)
- Reported rho, p, and n are consistent with the paper's Fig. 2a / Fig. 2b values
- Scatter plot reproduces the association shown in Fig. 2a / Fig. 2b and cross-checks against Supplementary Fig. 1A
- Signature genes are matched at the SYMBOL level against the dataset's gene rows before scoring the mean

## Limitations

- Bulk transcriptomic data limit the ability to fully separate cellular populations, so purity proxies are indirect
- Cross-dataset/platform aggregation is unreliable: 'inability to robustly merge datasets' across bulk RNA-seq and microarray platforms
- Including the 32 non-PDAC TCGA-PAAD samples (of 182 total) misleads signature validation; the Decision/whitelist filter is mandatory

## Evidence

- [readme] Decision/whitelist filter of TCGA-PAAD: [evidence span withheld — anchor status: not_found]
- [readme] Sample-level filtering removes labeled samples from analyses: "Checking a box in this list will remove all samples with that label from all analyses. This is important to do to ensure accruate analyses."
- [readme] Gene sets usable for correlations and visualization: "These gene sets can be used heatmapping, visualizing correlations, performing dimensionality reduction, and coloring differential expression plots."
- [intro] Only 150 of 182 TCGA samples are confirmed PDAC: "The mRNA dataset includes 182 samples, but only 150 have been confirmed histologically as PDAC"
- [methods] Package installation command: "devtools::install_github("rmoffitt/pdacR")"
- [discussion] Bulk data cannot fully separate cellular populations: "Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations"
