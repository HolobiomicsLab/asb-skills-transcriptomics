---
name: publication-figure-reproduction
description: Use when a target figure reports a specific statistic (e.g. a correlation
  between a geneset score and a sample-level covariate like VAF/purity) and the underlying
  parsed dataset and curated gene lists are available in pdacR's Data/ directory.
when_to_use_negative:
- The goal is to derive a new gene signature or subtype model rather than reproduce
  a published association — use pdacR's DE/clustering workflow instead.
- Working with the TCGA-PAAD dataset, where non-PDAC cases must first be filtered
  (select 'Decision' and keep only 'whitelist') before any signature-vs-purity comparison
  is meaningful.
- Single-cell data (e.g. rmoffitt/scOh Seurat objects) — the article notes bulk datasets
  are what limit cellular deconvolution, and the correlation-vs-VAF proxy logic is
  defined for bulk samples.
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: pdacR
  role: R package supplying the parsed Puleo 2018 microarray dataset, curated gene
    signature lists, and sample metadata needed to recompute the figure's signature
    scores and correlation
  repo: github.com/rmoffitt/pdacR
- name: R
  role: Runtime environment; cor.test(method = "spearman") computes the rank correlation
    reported in the figure
- name: ggplot2
  role: Renders the scatter plot of VAF versus exocrine score with a fitted trend
    line to visually reproduce Fig. 2b / Supplementary Fig. 1A
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:721e040f388288ce5e22af993b0a16390b91c9c43bc5c99b54494e79ee87119e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - ggplot2
  - pdacR
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# publication-figure-reproduction

## Summary

A workflow for reproducing a published PDAC figure (Puleo 2018 Fig. 2b / Supplementary Fig. 1A) by re-scoring the Collisson Exocrine signature on the 309-sample Puleo microarray dataset via pdacR and correlating it with predicted mean VAF as a tumor-purity proxy (reported Spearman rho = -0.221, p = 0.0003).

## When to use

Use when a target figure reports a specific statistic (e.g. a correlation between a geneset score and a sample-level covariate like VAF/purity) and the underlying parsed dataset and curated gene lists are available in pdacR's Data/ directory.

## When NOT to use

- The goal is to derive a new gene signature or subtype model rather than reproduce a published association — use pdacR's DE/clustering workflow instead.
- Working with the TCGA-PAAD dataset, where non-PDAC cases must first be filtered (select 'Decision' and keep only 'whitelist') before any signature-vs-purity comparison is meaningful.
- Single-cell data (e.g. rmoffitt/scOh Seurat objects) — the article notes bulk datasets are what limit cellular deconvolution, and the correlation-vs-VAF proxy logic is defined for bulk samples.

## Inputs

- pdacR parsed Puleo 2018 microarray dataset (list with $ex expression matrix and $sampInfo sample-level metadata)
- pdacR curated gene_lists object (Collisson Exocrine symbols) from the package Data/ directory
- Per-sample predicted mean variant allele frequency (VAF) values in $sampInfo
- Published reference figure: Puleo 2018 Fig. 2b and Supplementary Fig. 1A

## Outputs

- Spearman correlation estimate and p-value (target: rho = -0.221, p = 0.0003)
- ggplot2 scatter plot of predicted mean VAF versus exocrine signature score with fitted trend line

## How to apply

Load the parsed Puleo 2018 microarray dataset with pdacR::parse_Puleo_Array and the curated gene_lists from the package's Data/ directory. Subset the expression matrix ($ex) to the 309 FFPE primary PDAC samples, extract Collisson Exocrine symbols matched at the SYMBOL level, and score each sample as the mean expression of its genes. Pull predicted mean VAF from $sampInfo as the tumor purity proxy, then run cor.test(method = "spearman") in R between the two vectors. Render a ggplot2 scatter plot of VAF versus exocrine score with a fitted trend line, and cross-check against the published Fig. 2b and Supplementary Fig. 1A; accept the reproduction if the Spearman estimate (rho = -0.221, p = 0.0003 as reported) and plot geometry match.

## Related tools

- **pdacR** (R package supplying the parsed Puleo 2018 microarray dataset, curated gene signature lists, and sample metadata needed to recompute the figure's signature scores and correlation) — github.com/rmoffitt/pdacR
- **R** (Runtime environment; cor.test(method = "spearman") computes the rank correlation reported in the figure)
- **ggplot2** (Renders the scatter plot of VAF versus exocrine score with a fitted trend line to visually reproduce Fig. 2b / Supplementary Fig. 1A)

## Examples

```
pdacR::parse_Puleo_Array(); score <- colMeans(ex[gene_lists$Collisson_Exocrine, ]); cor.test(sampInfo$predicted_mean_VAF, score, method = "spearman")
```

## Evaluation signals

- Spearman coefficient is in the reported direction and magnitude (negative association; rho = -0.221, p = 0.0003 as reported for the Puleo 2018 dataset).
- Sample count of the subsetted $ex matrix equals 309 FFPE primary PDAC samples.
- Scatter plot geometry (negative slope of VAF vs exocrine score) matches Fig. 2b / Supplementary Fig. 1A of the source paper.
- Collisson Exocrine gene list resolves at the SYMBOL level and yields the expected gene count from the curated gene_lists object.
- cor.test is run with method = "spearman" (rank-based, so no normality assumption) and the p-value is reported alongside the estimate.

## Limitations

- pdacR datasets are bulk analyses: 'Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations', so VAF is only a proxy for purity.
- The predicted mean VAF is itself a prediction, so the correlation reflects classifier behavior as much as true tumor cellularity.
- Installation of pdacR is heavy ('there are quite a few datasets and substantial raw data within this package, so installation may take some time').

## Evidence

- [other] Compute the Spearman rank correlation between the exocrine signature score and predicted mean VAF using cor.test(method = "spearman") in R.: [evidence span withheld — anchor status: not_found]
- [readme] Data/ holds all parsed datasets, as well as the gene signatures object and PurIST classifier object.: "Data/ holds all parsed datasets, as well as the gene signatures object and PurIST classifier object. These datasets are used both in the app and in the [associated publication Torre-Healy et al. 2023]"
- [other] Render a scatter plot of VAF versus exocrine score with a fitted trend line (ggplot2) reproducing Fig. 2b, and cross-check against Supplementary Fig. 1A.: [evidence span withheld — anchor status: not_found]
- [other] subset the expression matrix ($ex) to the 309 FFPE primary PDAC samples: [evidence span withheld — anchor status: not_found]
