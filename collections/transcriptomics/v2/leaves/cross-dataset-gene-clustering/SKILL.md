---
name: cross-dataset-gene-clustering
description: Use when a published gene signature (e.g. Bailey ADEX) may be confounded
  by tissue-nonspecific or non-tumor genes, and you have ≥2 independently parsed bulk
  RNAseq datasets to test cross-dataset reproducibility of gene groupings.
when_to_use_negative:
- You want to cluster samples (tumors) rather than genes — pdacR's subtyping workflows
  handle sample-level clustering; this skill operates in gene-wise mode specifically
  to decompose signatures.
- Only one dataset is available — the classification of a gene as ACC-specific vs
  nonspecific depends on cross-dataset cluster membership, so a single-dataset run
  cannot support the conclusion.
- Single-cell resolution is required to separate tumor from microenvironment signals
  — the article notes bulk data limit the ability to fully separate cellular populations.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: pdacR
  role: R package supplying parsed PACA-AU and Moffitt RNAseq datasets and the gene_lists
    object with the Bailey signatures
  repo: https://github.com/rmoffitt/pdacR.git
- name: ConsensusClusterPlus
  role: Gene-wise consensus clustering (kmeans, euclidean distance) run separately
    on each dataset to group ADEX genes
- name: ggplot2
  role: Rendering the Fig. 3a,d panels (ADEX partition and Immunogenic overlap)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:916ab979d906706c679d56800e750852754dacefc1c83e5b1253536c8b2dd89a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rmoffitt/pdacR.git
  tools:
  - ConsensusClusterPlus
  - R
  - ggplot2
  - pdacR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cross-dataset-gene-clustering

## Summary

Runs gene-wise consensus clustering (ConsensusClusterPlus, kmeans with euclidean distance) independently on multiple PDAC RNAseq datasets to decompose a gene signature into subset-specific components, used to test whether the Bailey ADEX/Immunogenic signatures are confounded by non-PDAC-specific genes.

## When to use

Use when a published gene signature (e.g. Bailey ADEX) may be confounded by tissue-nonspecific or non-tumor genes, and you have ≥2 independently parsed bulk RNAseq datasets to test cross-dataset reproducibility of gene groupings.

## When NOT to use

- You want to cluster samples (tumors) rather than genes — pdacR's subtyping workflows handle sample-level clustering; this skill operates in gene-wise mode specifically to decompose signatures.
- Only one dataset is available — the classification of a gene as ACC-specific vs nonspecific depends on cross-dataset cluster membership, so a single-dataset run cannot support the conclusion.
- Single-cell resolution is required to separate tumor from microenvironment signals — the article notes bulk data limit the ability to fully separate cellular populations.

## Inputs

- PACA-AU RNAseq dataset in pdacR list format ($ex, $sampInfo, $featInfo, $metadata)
- Moffitt RNAseq dataset in pdacR list format ($ex, $sampInfo, $featInfo, $metadata)
- Bailey ADEX, Immunogenic, and pancreatic progenitor gene sets from the pdacR gene_lists object (gene SYMBOL identifiers)

## Outputs

- Three gene lists classifying each ADEX gene as ACC-specific, pancreas-nonspecific, or nonspecific
- Fig. 3a panel: ADEX signature partition by gene subset
- Fig. 3d panel: Immunogenic overlap with progenitor and ADEX signatures
- Results table of Immunogenic-progenitor and Immunogenic-ADEX overlap fractions (26% and 9%)

## How to apply

Load the parsed PACA-AU RNAseq and Moffitt RNAseq datasets from the pdacR data/ directory in the required $ex/$sampInfo/$featInfo/$metadata list format, and extract the Bailey ADEX, Immunogenic, and pancreatic progenitor gene sets from the pdacR gene_lists object (SYMBOL identifiers). Run ConsensusClusterPlus in gene-wise mode with the kmeans algorithm and euclidean distance separately on each dataset to group ADEX genes by their cross-dataset expression patterns. Classify each ADEX gene as ACC-specific, pancreas-nonspecific, or nonspecific based on consistent cluster membership across both datasets, and save the three gene lists as named output files. Finally compute percent overlap of the Immunogenic set against the progenitor and ADEX signatures (expect 26% and 9%) and render the Fig. 3a,d panels (ADEX partition and Immunogenic overlap) with an accompanying results table.

## Related tools

- **pdacR** (R package supplying parsed PACA-AU and Moffitt RNAseq datasets and the gene_lists object with the Bailey signatures) — https://github.com/rmoffitt/pdacR.git
- **ConsensusClusterPlus** (Gene-wise consensus clustering (kmeans, euclidean distance) run separately on each dataset to group ADEX genes)
- **ggplot2** (Rendering the Fig. 3a,d panels (ADEX partition and Immunogenic overlap))

## Examples

```
library(pdacR); library(ConsensusClusterPlus); run ConsensusClusterPlus(paca_au$ex[bailey_adex_genes,], maxK=3, clusterAlg="km", distance="euclidean")
```

## Evaluation signals

- Computed Immunogenic–pancreatic progenitor overlap fraction equals the paper-reported 26%.
- Computed Immunogenic–ADEX overlap fraction equals the paper-reported 9%.
- Each ADEX gene is assigned to exactly one of the three subsets (ACC-specific, pancreas-nonspecific, nonspecific) with cross-dataset cluster agreement.
- The three classified gene lists and both figure panels (Fig. 3a,d) exist as named output files.

## Limitations

- All pdacR datasets are bulk analyses, limiting the ability to fully separate cellular populations, so signature decomposition reflects aggregate expression signals.
- Datasets come from different platforms (bulk RNA-seq and microarrays), and robust cross-platform merging/batch correction is not possible in pdacR — clustering is therefore run separately per dataset rather than on pooled data.
- TCGA-PAAD contains 182 samples of which only 150 are histologically confirmed PDAC; uncurated sample sets can introduce non-specificity into gene signature analyses.

## Evidence

- [results] Gene-based consensus clustering of the PACA-AU RNAseq and Moffitt RNAseq datasets shows the Bailey ADEX gene set is comprised primarily of three gene subsets: generally nonspecific, pancreas nonspecific, and ACC-specific genes.: [evidence span withheld — anchor status: not_found]
- [abstract] Bioinformatic pitfalls litter public transcriptomic data, such as subtle inclusion of low-purity and non-adenocarcinoma cases. These pitfalls can introduce non-specificity to gene signatures without: "These pitfalls can introduce non-specificity to gene signatures without"
- [readme] Data/ holds all parsed datasets, as well as the gene signatures object and PurIST classifier object. These datasets are used both in the app and in the associated publication: "Data/ holds all parsed datasets, as well as the gene signatures object and PurIST classifier object."
- [discussion] Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations: "Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations"
