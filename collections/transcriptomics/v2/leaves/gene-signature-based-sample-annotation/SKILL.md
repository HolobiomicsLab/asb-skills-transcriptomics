---
name: gene-signature-based-sample-annotation
description: Use when bulk PDAC expression samples need subtype calls under one or
  more published signatures, or when concordance between Moffitt, Collisson, and Bailey
  subtype labels must be quantified per sample.
when_to_use_negative:
- When the input is single-cell RNA-seq — use the rmoffitt/scOh atlas and its pretrained
  RandomForest cell type classifier instead, since pdacR datasets are all bulk analyses
- When the dataset has not been filtered for non-PDAC or low-purity cases (e.g. full
  182-sample TCGA-PAAD), as uncurated samples introduce non-specificity to gene signatures
- When the goal is to relabel one scheme's calls directly as another (basal-like =
  squamous = QM); the signatures are not completely redundant and are confounded by
  purity and tissue-specificity
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: pdacR
  role: R package supplying the curated PDAC expression datasets, subtype gene signatures,
    PurIST classifier, sample-track filtering, and heatmap/cartesian/DE analysis interface
  repo: https://github.com/rmoffitt/pdacR.git
- name: R
  role: Runtime environment for pdacR; the package is an R library with wrapper functions
    in R/
- name: devtools
  role: Used to install pdacR from GitHub (devtools::install_github)
- name: shiny
  role: RShiny GUI (pdacR::pdacShiny) for point-and-click signature selection, sample
    filtering, and visualization
- name: PurIST
  role: Purity-independent single-sample subtype classifier included in pdacR for
    per-sample basal-like/classical calls
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:9393897a5cd9734991e586de8e78aaa0abbe03db2900b0fe7065953fd953b3fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rmoffitt/pdacR.git
  tools:
  - ConsensusClusterPlus
  - PurIST
  - R
  - devtools
  - pdacR
  - shiny
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# gene-signature-based-sample-annotation

## Summary

Assign PDAC samples to published molecular subtypes (Moffitt basal-like/classical, Collisson classical/QM/exocrine, Bailey progenitor/squamous/ADEX/immunogenic, or PurIST) by applying curated SYMBOL-level gene signatures to curated bulk expression datasets hosted in pdacR, then cross-tabulating concordance between schemes.

## When to use

Use when bulk PDAC expression samples need subtype calls under one or more published signatures, or when concordance between Moffitt, Collisson, and Bailey subtype labels must be quantified per sample.

## When NOT to use

- When the input is single-cell RNA-seq — use the rmoffitt/scOh atlas and its pretrained RandomForest cell type classifier instead, since pdacR datasets are all bulk analyses
- When the dataset has not been filtered for non-PDAC or low-purity cases (e.g. full 182-sample TCGA-PAAD), as uncurated samples introduce non-specificity to gene signatures
- When the goal is to relabel one scheme's calls directly as another (basal-like = squamous = QM); the signatures are not completely redundant and are confounded by purity and tissue-specificity

## Inputs

- Curated bulk PDAC expression dataset from pdacR (e.g. TCGA-PAAD with 182 samples reduced to 150 whitelisted PDAC via the 'Decision' track)
- Curated subtype gene signatures (Moffitt, Collisson, Bailey) as SYMBOL-level gene sets (gene_set_list.RData)
- PurIST single-sample classifier object
- Sample metadata / sample tracks for filtering (e.g. 'Decision' = whitelist)

## Outputs

- Per-sample subtype assignments under Moffitt (basal-like/classical), Collisson (classical/QM/exocrine), and Bailey (progenitor/squamous/ADEX/immunogenic) schemes
- Pairwise subtype concordance percentages (e.g. Moffitt-vs-Collisson and Moffitt-vs-Bailey overlap, expected ~60%)
- Heatmaps/cartesian plots of signature gene expression colored by subtype track

## How to apply

Load a curated bulk PDAC expression dataset from pdacR — e.g. TCGA-PAAD filtered to the 150 whitelisted PDAC samples by selecting the 'Decision' sample track and removing all variables except 'whitelist'. Load the curated subtype signatures (Moffitt, Collisson, Bailey) and the PurIST classifier object from Data/ (gene_set_list.RData). Restrict the expression matrix to each signature's genes and cluster samples (e.g. with ConsensusClusterPlus) to derive subtype groups, or use the single-sample PurIST classifier. Cross-tabulate Moffitt basal-like/classical calls against Collisson and Bailey labels and compute pairwise overlap percentages between analogous subtypes. Because the two-subtype and collapsed 3+ subtype signatures are not completely redundant, do not interchangeably relabel samples as basal-like, squamous, or QM — annotate under each scheme separately and report concordance.

## Related tools

- **pdacR** (R package supplying the curated PDAC expression datasets, subtype gene signatures, PurIST classifier, sample-track filtering, and heatmap/cartesian/DE analysis interface) — https://github.com/rmoffitt/pdacR.git
- **R** (Runtime environment for pdacR; the package is an R library with wrapper functions in R/)
- **devtools** (Used to install pdacR from GitHub (devtools::install_github))
- **shiny** (RShiny GUI (pdacR::pdacShiny) for point-and-click signature selection, sample filtering, and visualization)
- **PurIST** (Purity-independent single-sample subtype classifier included in pdacR for per-sample basal-like/classical calls)

## Examples

```
devtools::install_github("rmoffitt/pdacR"); pdacR::pdacShiny()
```

## Evaluation signals

- Sample count invariant: after applying the 'Decision'/whitelist filter, TCGA-PAAD must retain exactly 150 histologically confirmed PDAC samples out of 182
- Moffitt-vs-Collisson and Moffitt-vs-Bailey pairwise overlap percentages should approximate the ~60% concordance reported in Supplementary Fig. 1b,c
- Per-sample assignment table must label every sample under each scheme separately (basal-like/classical; classical/QM/exocrine; progenitor/squamous/ADEX/immunogenic) rather than collapsing 3+ subtype calls into two
- Heatmap of signature genes should show black-bar grouping of genes by their member gene set, with non-selected gene sets removed from the plot
- A writing a concordance CSV (e.g. subtype_concordance.csv) with per-sample labels and pairwise overlap percentages, matching the task's output specification

## Limitations

- Bulk transcriptomic data limit the ability to fully separate cellular populations, so subtype signatures reflect mixed tumor/stroma signals
- Datasets come from multiple platforms (bulk RNA-seq and microarrays), making cross-dataset aggregation and batch correction unreliable; pdacR removed data aggregation because it introduced more problems than it solved
- The two-subtype and collapsed 3+ subtype signatures are not completely redundant, and purity/tissue-specificity confounders differ between schemes
- Public PDAC transcriptomic data contains pitfalls such as subtle inclusion of low-purity and non-adenocarcinoma cases that can bias signature application without curation

## Evidence

- [readme] TCGA-PAAD has 182 samples, but only 150 are PDAC.: "TCGA-PAAD has 182 samples, but only 150 are PDAC. The remainder have been identified to be non-PDAC tumors in previous publications."
- [readme] To filter to the correct 150, select "Decision", and then remove all variables except "whitelist".: "To filter to the correct 150, select "Decision", and then remove all variables except "whitelist"."
- [intro] Given that the two-subtype and collapsed 3+ subtype signatures are not completely redundant, attempting to interchangeably label a subset of the data as basal-like, squamous, or QM is not equivalent.: "Given that the two-subtype and collapsed 3+ subtype signatures are not completely redundant, attempting to interchangeably label a subset of the data as basal-like, squamous, or QM is not equivalent."
- [methods] These gene sets can be used heatmapping, visualizing correlations, performing dimensionality reduction, and coloring differential expression plots.: "These gene sets can be used heatmapping, visualizing correlations, performing dimensionality reduction, and coloring differential expression plots."
- [intro] we have gathered and carefully annotated thirteen human PDAC datasets from the past decade for a thorough comparison of the performance of subtyping schemas in identifying only the tumor cells: "we have gathered and carefully annotated thirteen human PDAC datasets from the past decade for a thorough comparison of the performance of subtyping schemas in identifying only the tumor cells within"
