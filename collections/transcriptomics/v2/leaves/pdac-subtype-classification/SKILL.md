---
name: pdac-subtype-classification
description: Use when a bulk PDAC expression matrix (e.g. TCGA-PAAD) needs sample-level
  subtype calls under one or more of the Moffitt, Collisson, or Bailey schemas, or
  when comparing subtype assignments across these classification systems.
when_to_use_negative:
- Input is single-cell RNA-seq data — use the rmoffitt/scOh single-cell atlas instead;
  bulk signatures assume tumor/TME mixture of bulk samples
- The expression cohort contains non-PDAC samples (e.g. the 32 non-whitelist TCGA-PAAD
  samples) — these must be removed first or subtype calls and signature specificity
  are compromised
- Goal is single-sample clinical subtyping rather than cluster-based cohort analysis
  — the PurIST classifier in pdacR is the dedicated single-sample classifier
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2640
tools:
- name: pdacR
  role: Provides the curated PDAC bulk datasets, sample tracks ('Decision' whitelist),
    and SYMBOL-level subtype gene signature objects (Moffitt, Collisson, Bailey) used
    for subtype assignment
  repo: github.com/rmoffitt/pdacR
- name: R
  role: Package/environment in which pdacR and the clustering/concordance workflow
    are executed
  repo: github.com/rmoffitt/pdacR
- name: ConsensusClusterPlus
  role: Clusters samples restricted to each signature's genes to derive subtype groups
    prior to label assignment
- name: devtools
  role: Used to install pdacR from GitHub via devtools::install_github("rmoffitt/pdacR")
  repo: github.com/rmoffitt/pdacR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:1df0dc31fca82b92833e5ee358e649e782411724ce371bb108f8e84a081f673b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ConsensusClusterPlus
  - R
  - devtools
  - pdacR
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pdac-subtype-classification

## Summary

Classifies PDAC bulk tumor expression samples into the Moffitt basal-like/classical, Collisson classical/QM, and Bailey progenitor/squamous subtypes using curated SYMBOL-level gene signatures from the pdacR package, and quantifies cross-schema concordance (expected ~60% overlap between analogous subtype calls).

## When to use

Use when a bulk PDAC expression matrix (e.g. TCGA-PAAD) needs sample-level subtype calls under one or more of the Moffitt, Collisson, or Bailey schemas, or when comparing subtype assignments across these classification systems.

## When NOT to use

- Input is single-cell RNA-seq data — use the rmoffitt/scOh single-cell atlas instead; bulk signatures assume tumor/TME mixture of bulk samples
- The expression cohort contains non-PDAC samples (e.g. the 32 non-whitelist TCGA-PAAD samples) — these must be removed first or subtype calls and signature specificity are compromised
- Goal is single-sample clinical subtyping rather than cluster-based cohort analysis — the PurIST classifier in pdacR is the dedicated single-sample classifier

## Inputs

- Curated PDAC bulk expression matrix from pdacR (e.g. TCGA-PAAD filtered via the 'Decision' track to 150 whitelisted PDAC samples)
- SYMBOL-level curated subtype gene signatures: Moffitt, Collisson, Bailey (pdacR gene_sets object / gene_set_list.RData)
- Sample track metadata (e.g. 'Decision' whitelist labels) for filtering

## Outputs

- Per-sample subtype assignment table under Moffitt (basal-like/classical), Collisson (classical/QM/exocrine), and Bailey (progenitor/squamous/ADEX/immunogenic) schemes
- subtype_concordance.csv with pairwise overlap percentages between analogous subtypes (e.g. Moffitt-vs-Collisson, Moffitt-vs-Bailey)

## How to apply

First curate the cohort: for TCGA-PAAD, restrict to the 150 histologically confirmed PDAC samples by selecting the 'Decision' sample track and keeping only 'whitelist' samples, since the full mRNA dataset has 182 samples including 32 non-PDAC tumors. Load the curated subtype gene signatures (Moffitt, Collisson, Bailey) as SYMBOL-level gene sets via pdacR's gene_sets object (gene_set_list.RData / parse_gene_lists). For each signature, restrict the expression matrix to its genes and cluster samples with ConsensusClusterPlus to derive subtype groups, then assign per-sample labels: Moffitt basal-like/classical, Collisson classical/QM/exocrine, Bailey progenitor/squamous/ADEX/immunogenic. Cross-tabulate Moffitt calls against Collisson and Bailey labels and compute pairwise overlap percentages between analogous subtypes. Because the two-subtype and collapsed 3+ subtype signatures are not completely redundant, do not interchangeably relabel subtypes across schemas — report concordance explicitly instead.

## Related tools

- **pdacR** (Provides the curated PDAC bulk datasets, sample tracks ('Decision' whitelist), and SYMBOL-level subtype gene signature objects (Moffitt, Collisson, Bailey) used for subtype assignment) — github.com/rmoffitt/pdacR
- **R** (Package/environment in which pdacR and the clustering/concordance workflow are executed) — github.com/rmoffitt/pdacR
- **ConsensusClusterPlus** (Clusters samples restricted to each signature's genes to derive subtype groups prior to label assignment)
- **devtools** (Used to install pdacR from GitHub via devtools::install_github("rmoffitt/pdacR")) — github.com/rmoffitt/pdacR

## Examples

```
devtools::install_github("rmoffitt/pdacR"); library(pdacR); pdacShiny()  # select TCGA-PAAD, filter 'Decision'->'whitelist' (150 PDAC), then cluster signatures with ConsensusClusterPlus
```

## Evaluation signals

- TCGA-PAAD cohort size after 'Decision'/'whitelist' filtering equals 150 PDAC samples (out of 182 total)
- Moffitt-vs-Collisson and Moffitt-vs-Bailey overlap percentages for analogous subtypes approximate the ~60% concordance reported in Supplementary Fig. 1b,c
- All signature genes map to valid SYMBOLs present in the expression matrix before clustering
- Every sample receives a label under each schema applied, and the cross-tabulation covers all Moffitt basal-like/classical × Collisson/Bailey subtype pairs
- subtype_concordance.csv contains per-sample assignments plus pairwise concordance percentages matching the cross-tabulation counts

## Limitations

- Bulk transcriptomic data limit the ability to fully separate cellular populations, so subtype calls reflect mixed tumor/TME signals
- The two-subtype and collapsed 3+ subtype signatures (basal-like, squamous, QM) are not completely redundant; interchangeably relabeling them is not equivalent
- Datasets span multiple platforms (bulk RNA-seq and microarrays), making cross-platform merging/aggregation unreliable
- Validation of subtyping schemas is frequently performed on only one dataset; the ~60% concordance estimate should be interpreted with that in mind

## Evidence

- [supplementary] The Collisson classical/QM subtypes and the Bailey progenitor/squamous subtypes are highly concordant with the Moffitt classical/basal-like subtypes, respectively, showing approximately 60% overlap between the analogous subtype calls.: [evidence span withheld — anchor status: not_found]
- [readme] TCGA-PAAD 182 vs 150 samples: "For example, TCGA-PAAD has 182 samples, but only 150 are PDAC. The remainder have been identified to be non-PDAC tumors in previous publications. To filter to the correct 150, select "Decision", and"
- [intro] Signatures not redundant: "Given that the two-subtype and collapsed 3+ subtype signatures are not completely redundant, attempting to interchangeably label a subset of the data as basal-like, squamous, or QM is not equivalent."
- [abstract] multi-dataset analysis confirms two-subtype model: "we present a multi-dataset analysis of PDAC transcriptomics that confirms the basal-like/classical model over alternatives"
- [abstract] Bioinformatic pitfalls litter public transcriptomic data, such as subtle inclusion of low-purity and non-adenocarcinoma cases.: "Bioinformatic pitfalls litter public transcriptomic data, such as subtle inclusion of low-purity and non-adenocarcinoma cases."
