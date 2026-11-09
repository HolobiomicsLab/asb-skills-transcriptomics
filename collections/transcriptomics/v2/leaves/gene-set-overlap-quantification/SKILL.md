---
name: gene-set-overlap-quantification
description: Use when auditing whether a published PDAC signature (Bailey ADEX, Exocrine,
  Immunogenic) is confounded by non-tumor-specific genes, or when you need to reproduce
  signature-vs-signature overlap fractions across curated pdacR bulk RNAseq datasets.
when_to_use_negative:
- Input is single-cell data or another atlas (e.g., rmoffitt/scOh Seurat objects)
  rather than pdacR bulk transcriptomics — the skill's signatures and parsed $ex list
  format are defined for bulk datasets
- The goal is assigning subtype labels to samples (use the PurIST single-sample classifier
  or consensus clustering of samples) rather than measuring gene-set membership overlap
- The working dataset has not been curated (e.g., TCGA-PAAD still contains its 32
  non-PDAC samples), which would distort overlap fractions with non-adenocarcinoma
  signal
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: pdacR
  role: Source of parsed PACA-AU/Moffitt RNAseq datasets and the gene_lists object
    holding the Bailey ADEX, Immunogenic, and pancreatic progenitor signatures used
    for overlap computation
  repo: github.com/rmoffitt/pdacR
- name: ConsensusClusterPlus
  role: Run in gene-wise mode with kmeans and euclidean distance to cluster ADEX genes
    into ACC-specific, pancreas-nonspecific, and nonspecific subsets
- name: R
  role: Implementation environment for loading pdacR data, computing overlap fractions,
    and rendering figure panels
  repo: github.com/rmoffitt/pdacR
- name: ggplot2
  role: Rendering the Fig. 3a,d panels (ADEX gene-set partition and Immunogenic overlap
    visualization)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:d411ddc517d279c4cb0b69e1492de4e4a7d1835b4bb9c9e2063c4ea2ef130b78
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ConsensusClusterPlus
  - R
  - ggplot2
  - pdacR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# gene-set-overlap-quantification

## Summary

Quantify the percent overlap between curated PDAC gene signatures (e.g., Bailey Immunogenic, ADEX, pancreatic progenitor) using the pdacR gene_lists object, to test whether published signatures are confounded by non-PDAC-specific genes.

## When to use

Use when auditing whether a published PDAC signature (Bailey ADEX, Exocrine, Immunogenic) is confounded by non-tumor-specific genes, or when you need to reproduce signature-vs-signature overlap fractions across curated pdacR bulk RNAseq datasets.

## When NOT to use

- Input is single-cell data or another atlas (e.g., rmoffitt/scOh Seurat objects) rather than pdacR bulk transcriptomics — the skill's signatures and parsed $ex list format are defined for bulk datasets
- The goal is assigning subtype labels to samples (use the PurIST single-sample classifier or consensus clustering of samples) rather than measuring gene-set membership overlap
- The working dataset has not been curated (e.g., TCGA-PAAD still contains its 32 non-PDAC samples), which would distort overlap fractions with non-adenocarcinoma signal

## Inputs

- parsed PACA-AU RNAseq dataset (pdacR $ex/$sampInfo/$featInfo/$metadata list)
- parsed Moffitt RNAseq dataset (pdacR $ex/$sampInfo/$featInfo/$metadata list)
- pdacR gene_lists object with Bailey ADEX, Immunogenic, and pancreatic progenitor signatures (gene SYMBOLs)

## Outputs

- percent-overlap values of the Immunogenic set vs pancreatic progenitor (26%) and vs ADEX (9%)
- three ADEX gene-subset lists: ACC-specific, pancreas-nonspecific, nonspecific
- results table and rendered Fig. 3a (ADEX partition) and Fig. 3d (Immunogenic overlap) panels

## How to apply

Load the parsed PACA-AU RNAseq and Moffitt RNAseq datasets from the pdacR data/ directory, keeping the required $ex/$sampInfo/$featInfo/$metadata list structure, and filter TCGA-PAAD to the 150 histologically confirmed PDAC samples via the 'Decision' whitelist variable. Extract the Bailey ADEX, Immunogenic, and pancreatic progenitor signatures from the pdacR gene_lists object (gene SYMBOL identifiers, as parsed in R/parse_gene_lists.R). Compute the percent overlap of the Immunogenic gene set against the pancreatic progenitor and ADEX signatures as the fraction of Immunogenic genes present in each reference set; the reported values are 26% (vs progenitor) and 9% (vs ADEX). For the ADEX confounding analysis, run ConsensusClusterPlus in gene-wise mode with kmeans and euclidean distance on each dataset separately, then classify each ADEX gene as ACC-specific, pancreas-nonspecific, or nonspecific from its cross-dataset cluster membership and save the three gene lists as named outputs.

## Related tools

- **pdacR** (Source of parsed PACA-AU/Moffitt RNAseq datasets and the gene_lists object holding the Bailey ADEX, Immunogenic, and pancreatic progenitor signatures used for overlap computation) — github.com/rmoffitt/pdacR
- **ConsensusClusterPlus** (Run in gene-wise mode with kmeans and euclidean distance to cluster ADEX genes into ACC-specific, pancreas-nonspecific, and nonspecific subsets)
- **R** (Implementation environment for loading pdacR data, computing overlap fractions, and rendering figure panels) — github.com/rmoffitt/pdacR
- **ggplot2** (Rendering the Fig. 3a,d panels (ADEX gene-set partition and Immunogenic overlap visualization))

## Examples

```
devtools::install_github("rmoffitt/pdacR")
```

## Evaluation signals

- Computed Immunogenic-vs-pancreatic-progenitor overlap equals the paper-reported 26% and Immunogenic-vs-ADEX overlap equals 9%
- Three ADEX gene subsets (ACC-specific, pancreas-nonspecific, nonspecific) are generated as named output files
- Fig. 3a and Fig. 3d panels and the accompanying results table are produced as named outputs
- All input signatures resolve cleanly against the gene SYMBOL identifiers in the pdacR gene_lists object
- TCGA-PAAD-style datasets are filtered to the 150 whitelist PDAC samples before overlap computation

## Limitations

- Bulk transcriptomic data limit the ability to fully separate cellular populations, so signature confounding reflects bulk composition effects
- Datasets come from a variety of platforms (bulk RNA sequencing and microarrays), and robust cross-platform merging is not possible; the skill must be run per-dataset rather than on aggregated data
- Public PDAC data contains low-purity and non-adenocarcinoma cases (e.g., 32 of 182 TCGA-PAAD samples are non-PDAC) that must be filtered out or non-specificity will contaminate the overlap estimates

## Evidence

- [readme] To filter to the correct 150, select "Decision", and then remove all variables except "whitelist".: "TCGA-PAAD has 182 samples, but only 150 are PDAC. The remainder have been identified to be non-PDAC tumors in previous publications. To filter to the correct 150, select "Decision", and then remove"
- [readme] These gene sets can be used heatmapping, visualizing correlations, performing dimensionality reduction, and coloring differential expression plots.: "These gene sets can be used heatmapping, visualizing correlations, performing dimensionality reduction, and coloring differential expression plots."
- [abstract] These pitfalls can introduce non-specificity to gene signatures without curation.: "Bioinformatic pitfalls litter public transcriptomic data, such as subtle inclusion of low-purity and non-adenocarcinoma cases. These pitfalls can introduce non-specificity to gene signatures without"
- [intro] we have created an R package, termed pdacR, for public use: "we have created an R package, termed pdacR, for public use"
