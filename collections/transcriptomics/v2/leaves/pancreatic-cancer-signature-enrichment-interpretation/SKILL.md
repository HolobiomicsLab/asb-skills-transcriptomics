---
name: pancreatic-cancer-signature-enrichment-interpretation
description: Use when interpreting the cellular origin of PDAC gene signatures across
  sample compartments (primary PDAC, PDX, CAF lines, cell lines, normal pancreas,
  metastasis) in bulk transcriptomic datasets.
when_to_use_negative:
- When the goal is to separate tumor from stromal signals at single-cell resolution
  — the pdacR datasets are bulk analyses and cannot fully separate cellular populations
- When samples lack tissue-of-origin annotation (e.g. unfiltered TCGA-PAAD with non-PDAC
  cases), since the Wilcoxon comparison depends on correct sample tracks
- When merging scores across the microarray and RNAseq datasets, since cross-platform
  aggregation and batch correction are not supported
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: pdacR
  role: Source of the parsed Moffitt 2015 expression datasets (.RData), sampInfo metadata,
    and curated gene signature objects used for scoring ADEX/Exocrine/Immunogenic
  repo: github.com/rmoffitt/pdacR
- name: R
  role: Language in which the pdacR package, gene-list parsing (R/parse_gene_lists.R),
    z-scoring, and Wilcoxon tests are implemented
- name: devtools
  role: Used to install the pdacR package from GitHub to access its data and wrapper
    functions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:6a7318babdc8e411151641a0e59d91a8c1c962682c9787b638fa556790513a1e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - devtools
  - pdacR
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# pancreatic-cancer-signature-enrichment-interpretation

## Summary

Scores curated PDAC gene signatures (ADEX, Exocrine, Immunogenic) per sample across the Moffitt 2015 microarray and RNAseq datasets and uses Wilcoxon rank-sum tests to attribute signature expression to tissue/cell types, revealing whether expression is tumor-cell-intrinsic or derived from normal pancreas compartments.

## When to use

Use when interpreting the cellular origin of PDAC gene signatures across sample compartments (primary PDAC, PDX, CAF lines, cell lines, normal pancreas, metastasis) in bulk transcriptomic datasets.

## When NOT to use

- When the goal is to separate tumor from stromal signals at single-cell resolution — the pdacR datasets are bulk analyses and cannot fully separate cellular populations
- When samples lack tissue-of-origin annotation (e.g. unfiltered TCGA-PAAD with non-PDAC cases), since the Wilcoxon comparison depends on correct sample tracks
- When merging scores across the microarray and RNAseq datasets, since cross-platform aggregation and batch correction are not supported

## Inputs

- Parsed Moffitt 2015 microarray and RNAseq expression datasets (.RData objects from the pdacR data/ directory)
- sampInfo sample-level metadata annotating tissue of origin (primary PDAC, PDX, CAF line, cell line, normal pancreas, metastasis)
- Curated gene signature lists (ADEX, Exocrine, Immunogenic) in gene SYMBOL format from R/parse_gene_lists.R

## Outputs

- Per-sample signature scores plus Wilcoxon statistics and p-values (adex_exocrine_immunogenic_scores.csv)
- Grouped boxplots of signature scores by tissue type for the microarray and RNAseq datasets

## How to apply

Load the Moffitt 2015 microarray and RNAseq datasets as parsed .RData objects from the pdacR data/ directory together with their sampInfo metadata annotating tissue of origin. Take the ADEX, Exocrine, and Immunogenic gene sets from the curated signature lists built by R/parse_gene_lists.R, confirming identifiers are gene SYMBOLs. For each dataset, z-score each gene within the $ex genes-by-samples matrix and compute each sample's signature score as the mean scaled expression of the genes in that set — this mirrors the 'Expression Signature' computation pdacR's GUI applies for coloring and dot plots. Group samples by their tissue-of-origin sample track and run Wilcoxon rank-sum tests for normal pancreas vs PDAC and primary PDAC vs CAF lines. Interpret a significant enrichment of ADEX/Exocrine/Immunogenic in normal pancreata as evidence for a pancreas-cell-intrinsic, not tumor-cell-intrinsic, origin of the signature.

## Related tools

- **pdacR** (Source of the parsed Moffitt 2015 expression datasets (.RData), sampInfo metadata, and curated gene signature objects used for scoring ADEX/Exocrine/Immunogenic) — github.com/rmoffitt/pdacR
- **R** (Language in which the pdacR package, gene-list parsing (R/parse_gene_lists.R), z-scoring, and Wilcoxon tests are implemented)
- **devtools** (Used to install the pdacR package from GitHub to access its data and wrapper functions)

## Examples

```
devtools::install_github("rmoffitt/pdacR"); scores <- colMeans(t(scale(t(dat$ex)))[sig$ADEX, ])
```

## Evaluation signals

- Normal pancreata show significantly higher ADEX/Exocrine/Immunogenic scores than PDAC tumors (p < 0.0001) in both the microarray and RNAseq datasets
- Primary PDACs show higher ADEX/Exocrine/Immunogenic scores than CAF lines at p < 0.0001 in both datasets
- adex_exocrine_immunogenic_scores.csv contains per-sample scores, Wilcoxon statistics, and p-values for every tissue group in each dataset
- Signature scores match the expected scale of means of z-scored genes (mean ~0 per dataset), confirming correct centering/scaling of the $ex matrix
- Gene identifiers in the signature lists are gene SYMBOLs and resolve against the dataset row names before scoring

## Limitations

- All pdacR datasets are bulk analyses, limiting the ability to fully separate cellular populations driving signature expression
- Datasets span multiple platforms (bulk RNA-seq and microarrays), so aggregation and batch correction across datasets is unreliable
- Non-adenocarcinoma or low-purity samples must be filtered out (e.g. TCGA-PAAD: 182 samples, only 150 confirmed PDAC) or signature interpretation is confounded

## Evidence

- [abstract] we have created pdacR ( http://pdacR.bmi.stonybrook.edu , github.com/rmoffitt/pdacR), an open-source software package and web-tool with annotated datasets: "we have created pdacR ( http://pdacR.bmi.stonybrook.edu , github.com/rmoffitt/pdacR), an open-source software package and web-tool with annotated datasets"
- [discussion] we developed an R package termed pdacR that houses carefully organized and annotated public PDAC transcriptomics data: "we developed an R package termed pdacR that houses carefully organized and annotated public PDAC transcriptomics data for ease of access and heightened transparency"
- [readme] Data/ holds all parsed datasets, as well as the gene signatures object and PurIST classifier object.: "Data/ holds all parsed datasets, as well as the gene signatures object and PurIST classifier object. These datasets are used both in the app and in the [associated publication Torre-Healy et al. 2023]"
- [readme] These gene sets can be used heatmapping, visualizing correlations, performing dimensionality reduction, and coloring differential expression plots.: "These gene sets can be used heatmapping, visualizing correlations, performing dimensionality reduction, and coloring differential expression plots."
- [discussion] Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations: "Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations"
