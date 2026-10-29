---
name: gene-signature-specificity-partitioning
description: Use when a curated gene signature needs to be audited for tissue- or
  tumor-specificity confounders — e.g., checking if ADEX/Immunogenic genes are driven
  by non-PDAC or adrenal (ACC) signal before using the signature for subtyping.
when_to_use_negative:
- When the goal is sample-level tumor subtyping (e.g., basal-like/classical assignment
  or PurIST prediction) rather than auditing which genes in a signature are tissue-
  or tumor-specific
- When working with single-cell data — the article notes the parsed single-cell atlas
  (rmoffitt/scOh) lacks a web interface and that bulk datasets, not scRNA, are what
  pdacR houses for this analysis
- When datasets are not curated for sample composition; including non-adenocarcinoma
  cases (e.g., the 32 non-PDAC samples in TCGA-PAAD's 182) would corrupt the specificity
  clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: pdacR
  role: Supplies the parsed PACA-AU and Moffitt RNAseq datasets ($ex/$sampInfo/$featInfo/$metadata
    format) and the curated gene_lists object (Bailey ADEX, Immunogenic, pancreatic
    progenitor signatures in gene SYMBOLs)
  repo: github.com/rmoffitt/pdacR
- name: R
  role: Language in which the clustering, overlap computation, and figure generation
    are scripted; pdacR is installed as an R package
  repo: github.com/rmoffitt/pdacR
- name: ConsensusClusterPlus
  role: Performs gene-wise consensus clustering (kmeans algorithm, euclidean distance)
    separately on each dataset to group ADEX genes by specificity class
- name: ggplot2
  role: Renders the Fig. 3a,d panels showing the ADEX gene partition and the Immunogenic
    signature overlaps
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:d068855f5a4a9c72a9db0614c501ee42f65a1d0c069596913cdb436d2e2cd3d7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ConsensusClusterPlus
  - R
  - ggplot2
  - pdacR
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gene-signature-specificity-partitioning

## Summary

Tests whether a published PDAC gene signature (e.g., the Bailey ADEX or Immunogenic sets) is confounded by non-PDAC-specific genes by running gene-wise consensus clustering on curated bulk RNA-seq datasets and partitioning signature genes into specificity classes (ACC-specific, pancreas-nonspecific, nonspecific), then quantifying overlap against other signatures.

## When to use

Use when a curated gene signature needs to be audited for tissue- or tumor-specificity confounders — e.g., checking if ADEX/Immunogenic genes are driven by non-PDAC or adrenal (ACC) signal before using the signature for subtyping.

## When NOT to use

- When the goal is sample-level tumor subtyping (e.g., basal-like/classical assignment or PurIST prediction) rather than auditing which genes in a signature are tissue- or tumor-specific
- When working with single-cell data — the article notes the parsed single-cell atlas (rmoffitt/scOh) lacks a web interface and that bulk datasets, not scRNA, are what pdacR houses for this analysis
- When datasets are not curated for sample composition; including non-adenocarcinoma cases (e.g., the 32 non-PDAC samples in TCGA-PAAD's 182) would corrupt the specificity clustering

## Inputs

- Parsed PACA-AU RNAseq dataset in pdacR $ex/$sampInfo/$featInfo/$metadata list format
- Parsed Moffitt RNAseq dataset in pdacR $ex/$sampInfo/$featInfo/$metadata list format
- pdacR gene_lists object with Bailey ADEX, Immunogenic, and pancreatic progenitor signatures (gene SYMBOL identifiers)

## Outputs

- Three gene lists partitioning ADEX signature genes into ACC-specific, pancreas-nonspecific, and nonspecific subsets
- Computed percent-overlap values of Immunogenic vs. pancreatic progenitor (expected 26%) and Immunogenic vs. ADEX (expected 9%)
- Rendered Fig. 3a,d panels (ADEX partition and Immunogenic overlap) and an accompanying results table

## How to apply

Load the parsed PACA-AU RNAseq and Moffitt RNAseq datasets from the pdacR data/ directory in the required $ex/$sampInfo/$featInfo/$metadata list format, and extract the Bailey ADEX, Immunogenic, and pancreatic progenitor signatures (gene SYMBOL identifiers) from the pdacR gene_lists object as parsed by R/parse_gene_lists.R. Run ConsensusClusterPlus in gene-wise mode with the kmeans algorithm and euclidean distance separately on each dataset so that individual genes — not samples — are the clustering units; this reveals which ADEX genes co-cluster with adrenal/ACC-specific versus pancreas-expressed groups across two independent cohorts. Classify each ADEX gene as ACC-specific, pancreas-nonspecific, or nonspecific based on its cross-dataset cluster membership, and save the three gene lists as named outputs. Finally, compute the percent overlap of the Immunogenic gene set against the pancreatic progenitor and ADEX signatures and render the Fig. 3a,d panels (ADEX partition and Immunogenic overlap) with ggplot2.

## Related tools

- **pdacR** (Supplies the parsed PACA-AU and Moffitt RNAseq datasets ($ex/$sampInfo/$featInfo/$metadata format) and the curated gene_lists object (Bailey ADEX, Immunogenic, pancreatic progenitor signatures in gene SYMBOLs)) — github.com/rmoffitt/pdacR
- **R** (Language in which the clustering, overlap computation, and figure generation are scripted; pdacR is installed as an R package) — github.com/rmoffitt/pdacR
- **ConsensusClusterPlus** (Performs gene-wise consensus clustering (kmeans algorithm, euclidean distance) separately on each dataset to group ADEX genes by specificity class)
- **ggplot2** (Renders the Fig. 3a,d panels showing the ADEX gene partition and the Immunogenic signature overlaps)

## Examples

```
devtools::install_github("rmoffitt/pdacR")
```

## Evaluation signals

- The computed Immunogenic vs. pancreatic progenitor overlap fraction equals the paper-reported 26%
- The computed Immunogenic vs. ADEX overlap fraction equals the paper-reported 9%
- Three distinct ADEX gene subsets (ACC-specific, pancreas-nonspecific, nonspecific) are produced as named output gene lists
- The ADEX partition shows the set is comprised primarily of three gene subsets: generally nonspecific, pancreas nonspecific, and ACC-specific genes
- The Fig. 3a,d panels and the accompanying results table are generated as named output files

## Limitations

- Bulk transcriptomic data limit the ability to fully separate cellular populations, so specificity assignments reflect bulk signal, not single-cell resolution
- Datasets come from a variety of platforms (bulk RNA-seq and microarrays), and pdacR found the inability to robustly merge datasets across platforms — so clustering must be run separately per dataset rather than pooled
- Gene sets such as ADEX are not completely redundant with collapsed 3+ subtype signatures, so the specificity partition is not interchangeable with squamous/QM labels
- Public PDAC transcriptomic data contains pitfalls such as inclusion of low-purity and non-adenocarcinoma cases (e.g., TCGA-PAAD 182 samples with only 150 PDAC), which introduce non-specificity if not filtered

## Evidence

- [results] Gene-based consensus clustering of the PACA-AU RNAseq and Moffitt RNAseq datasets shows the Bailey ADEX gene set is comprised primarily of three gene subsets: generally nonspecific, pancreas nonspecific, and ACC-specific genes.: [evidence span withheld — anchor status: not_found]
- [abstract] Bioinformatic pitfalls litter public transcriptomic data, such as subtle inclusion of low-purity and non-adenocarcinoma cases. These pitfalls can introduce non-specificity to gene signatures without: "Bioinformatic pitfalls litter public transcriptomic data, such as subtle inclusion of low-purity and non-adenocarcinoma cases. These pitfalls can introduce non-specificity to gene signatures"
- [intro] The mRNA dataset includes 182 samples, but only 150 have been confirmed histologically as PDAC: "The mRNA dataset includes 182 samples, but only 150 have been confirmed histologically as PDAC"
- [discussion] Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations: "Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations"
- [methods] devtools::install_github("rmoffitt/pdacR"): "devtools::install_github("rmoffitt/pdacR")"
