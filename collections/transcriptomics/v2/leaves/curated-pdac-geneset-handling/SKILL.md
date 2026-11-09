---
name: curated-pdac-geneset-handling
description: Use when analyzing public PDAC bulk RNA-seq/microarray data with curated
  subtype signatures (ADEX, Immunogenic, basal-like/classical) and you must avoid
  contaminating samples (e.g., TCGA-PAAD's 32 non-PDAC cases) or signature non-specificity.
when_to_use_negative:
- Single-cell RNA-seq work — pdacR is bulk-only and its GUI cannot handle scRNA computational
  burden; use the separate rmoffitt/scOh atlas instead
- When you need to robustly merge or batch-correct datasets across platforms (bulk
  RNA-seq vs. microarrays) — the authors removed data aggregation because it 'introduced
  more problems than it solved'
- Datasets already containing only validated tumor-cell samples with no risk of non-adenocarcinoma
  or low-purity contamination
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: pdacR
  role: R package providing parsed PDAC datasets, the curated gene-signature object,
    PurIST classifier, and Shiny app for gene-set-based analyses
  repo: https://github.com/rmoffitt/pdacR
- name: ConsensusClusterPlus
  role: Runs gene-wise kmeans clustering with euclidean distance to decompose the
    ADEX signature into ACC-specific, pancreas-nonspecific, and nonspecific subsets
- name: shiny
  role: RShiny app (pdacShiny) exposing dataset selection, sample filtering, and gene-set
    selection for heatmapping/clustering/DE/survival
- name: devtools
  role: Installation of pdacR from GitHub
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:9d1a18ac7e79b57b3339218cff003dd15dda882939d469633faf4b8dc225d656
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rmoffitt/pdacR
  tools:
  - ConsensusClusterPlus
  - R
  - devtools
  - ggplot2
  - pdacR
  - shiny
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# curated-pdac-geneset-handling

## Summary

Load pdacR's parsed, annotated PDAC transcriptomic datasets and curated gene-signature object (Bailey ADEX, Immunogenic, pancreatic progenitor, PurIST), filter out non-adenocarcinoma samples, and apply the gene sets consistently across heatmapping, clustering, dimensionality reduction, DE, and signature-overlap analyses.

## When to use

Use when analyzing public PDAC bulk RNA-seq/microarray data with curated subtype signatures (ADEX, Immunogenic, basal-like/classical) and you must avoid contaminating samples (e.g., TCGA-PAAD's 32 non-PDAC cases) or signature non-specificity.

## When NOT to use

- Single-cell RNA-seq work — pdacR is bulk-only and its GUI cannot handle scRNA computational burden; use the separate rmoffitt/scOh atlas instead
- When you need to robustly merge or batch-correct datasets across platforms (bulk RNA-seq vs. microarrays) — the authors removed data aggregation because it 'introduced more problems than it solved'
- Datasets already containing only validated tumor-cell samples with no risk of non-adenocarcinoma or low-purity contamination

## Inputs

- pdacR parsed dataset objects in the $ex/$sampInfo/$featInfo/$metadata list format (e.g., PACA-AU RNAseq, Moffitt RNAseq)
- pdacR gene_lists object of curated gene signatures (Bailey ADEX, Immunogenic, pancreatic progenitor; gene SYMBOL identifiers)
- PurIST classifier object from the pdacR Data/ directory
- Sample metadata tracks (e.g., TCGA-PAAD 'Decision' whitelist labels) used for sample filtering

## Outputs

- Whitelist-filtered expression matrices (TCGA-PAAD: 150 confirmed PDAC samples)
- Selected gene-set subsets for heatmapping, clustering, DE, and dimensionality reduction
- Signature-overlap fractions (Immunogenic vs. pancreatic progenitor = 26%; vs. ADEX = 9%)
- Partitioned ADEX gene lists (ACC-specific, pancreas-nonspecific, generally nonspecific) from gene-based consensus clustering

## How to apply

Install pdacR via devtools::install_github("rmoffitt/pdacR") and load parsed datasets from the data/ directory, which follow the $ex/$sampInfo/$featInfo/$metadata list format. Before any analysis, filter samples using dataset-specific metadata — for TCGA-PAAD select the 'Decision' track and keep only 'whitelist' samples so 182 samples reduce to the 150 histologically confirmed PDAC cases. Extract gene signatures from the pdacR gene_lists object (HGNC gene SYMBOL identifiers, populated by R/parse_gene_lists.R), optionally augmenting with 'user selected genes' entered as comma/space-separated sets. The selected sets then drive heatmapping (with black gene-set bars and set deselection to view overlaps), ConsensusClusterPlus gene-wise clustering (kmeans, euclidean distance), Cartesian/tsne projection, survival, and DE analyses, and support quantitative signature-overlap computations (e.g., Immunogenic vs. pancreatic progenitor = 26%, vs. ADEX = 9%). Rationale: uncurated public data contains low-purity and non-adenocarcinoma cases that introduce non-specificity into gene signatures, so curation and whitelist filtering precede all downstream use.

## Related tools

- **pdacR** (R package providing parsed PDAC datasets, the curated gene-signature object, PurIST classifier, and Shiny app for gene-set-based analyses) — https://github.com/rmoffitt/pdacR
- **ConsensusClusterPlus** (Runs gene-wise kmeans clustering with euclidean distance to decompose the ADEX signature into ACC-specific, pancreas-nonspecific, and nonspecific subsets)
- **shiny** (RShiny app (pdacShiny) exposing dataset selection, sample filtering, and gene-set selection for heatmapping/clustering/DE/survival)
- **devtools** (Installation of pdacR from GitHub)

## Examples

```
devtools::install_github("rmoffitt/pdacR"); pdacR::pdacShiny()
```

## Evaluation signals

- TCGA-PAAD sample count drops from 182 to exactly 150 after applying the 'Decision'/'whitelist' filter
- Immunogenic-vs-progenitor overlap equals the reported 26% and Immunogenic-vs-ADEX overlap equals 9%
- ADEX genes classify into the three expected subsets (ACC-specific, pancreas-nonspecific, generally nonspecific) with cross-dataset consistency between PACA-AU and Moffitt RNAseq
- All gene identifiers in extracted signatures are HGNC symbols consistent with R/parse_gene_lists.R output
- Resulting figures (Fig. 3a,d) and results table render as named output files

## Limitations

- Bulk transcriptomic data limit the ability to fully separate cellular populations, so signatures may carry TME (fibroblast, endocrine, exocrine, immune) signals
- Datasets span multiple platforms (bulk RNA-seq and microarrays) and cannot be robustly merged or batch-corrected; aggregation was removed from the package
- The accompanying single-cell atlas (rmoffitt/scOh) lacks a web interface, and scRNA-seq remains prohibitively expensive for clinical application

## Evidence

- [abstract] Bioinformatic pitfalls litter public transcriptomic data, such as subtle inclusion of low-purity and non-adenocarcinoma cases.: "Bioinformatic pitfalls litter public transcriptomic data, such as subtle inclusion of low-purity and non-adenocarcinoma cases. These pitfalls can introduce non-specificity to gene signatures without"
- [methods] TCGA-PAAD sample filtering: "For example, TCGA-PAAD has 182 samples, but only 150 are PDAC."
- [readme] pdacR data directory contents: "Data/ holds all parsed datasets, as well as the gene signatures object and PurIST classifier object."
- [intro] Filtering rationale: "The mRNA dataset includes 182 samples, but only 150 have been confirmed histologically as PDAC"
- [discussion] Aggregation limitation: "we found that aggregation introduced more problems than it solved"
