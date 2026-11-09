---
name: paired-sample-matching-across-datasets
description: Use when PDAC expression datasets contain patient-matched primary tumors,
  metastases, and/or derived cell lines (e.g., PACA-AU paired samples) and you need
  to compare signature scores across matched sample classes rather than across unrelated
  samples.
when_to_use_negative:
- When datasets have no patient-level pairing information in sampInfo — pdacR notes
  the 'inability to robustly merge datasets', so cross-platform matching without sample
  metadata is not defensible
- When the goal is cohort-level subtype classification (e.g., PurIST) rather than
  within-patient comparison of signature retention/loss
- When samples include low-purity or non-adenocarcinoma cases (e.g., the 32 non-PDAC
  TCGA-PAAD samples) that have not been filtered out, since these confound signature
  scoring
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: pdacR
  role: Supplies the parsed PACA-AU RNAseq/microarray expression matrices, sampInfo
    metadata for paired-sample identification, ICGC_PARSE_Nones filter flags, and
    the gene_lists object with the ADEX/Immunogenic/Exocrine signatures
  repo: https://github.com/rmoffitt/pdacR
- name: R
  role: Runtime environment in which the pdacR package objects (.rds) are loaded,
    samples are paired via sampInfo, and z-scored signature scores are computed
  repo: https://github.com/rmoffitt/pdacR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:f90703f27d4fa1429e20140f2b45ac8abf79b73244df3d480d51a1cd7a2446cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rmoffitt/pdacR
  tools:
  - R
  - pdacR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# paired-sample-matching-across-datasets

## Summary

Identify matched samples (primary tumor, metastatic tumor, and derived cell line from the same patient) within and between the ICGC PACA-AU RNAseq and microarray expression sets using pdacR's parsed sampInfo metadata, enabling longitudinal/paired comparisons of gene-signature retention or loss.

## When to use

Use when PDAC expression datasets contain patient-matched primary tumors, metastases, and/or derived cell lines (e.g., PACA-AU paired samples) and you need to compare signature scores across matched sample classes rather than across unrelated samples.

## When NOT to use

- When datasets have no patient-level pairing information in sampInfo — pdacR notes the 'inability to robustly merge datasets', so cross-platform matching without sample metadata is not defensible
- When the goal is cohort-level subtype classification (e.g., PurIST) rather than within-patient comparison of signature retention/loss
- When samples include low-purity or non-adenocarcinoma cases (e.g., the 32 non-PDAC TCGA-PAAD samples) that have not been filtered out, since these confound signature scoring

## Inputs

- Parsed ICGC PACA-AU RNAseq expression matrix (.rds, genes x samples) with matching sampInfo metadata
- Parsed ICGC PACA-AU microarray expression matrix (.rds, genes x samples) with matching sampInfo metadata
- pdacR gene_lists object (Bailey ADEX, Bailey Immunogenic, Collisson Exocrine signatures) from R/parse_gene_lists.R
- ICGC_PARSE_Nones flags / sample-level metadata used for sample filtering and pair identification

## Outputs

- Per-sample gene set score table (samples x 3 signatures) with class labels (primary/metastatic/cell line) and platform annotation, written to icgc_gene_set_scores.csv
- Paired sample assignments (primary-primary, primary-metastatic, primary-cell line) across PACA-AU RNAseq and microarray platforms

## How to apply

Load the parsed ICGC PACA-AU RNAseq and microarray expression matrices (genes x samples) with their matching sampInfo from the pdacR data directory (populated by R/parse_gene_lists.R and the .rds parse_ICGC-derived objects). Use the sampInfo sample-level metadata to identify pair relationships — primary-primary, primary-metastatic, and primary-cell line — both within each platform and across the two PACA-AU platforms, first excluding any samples flagged by ICGC_PARSE_Nones (pdacR's curated whitelist-style filtering, analogous to removing non-PDAC 'Decision' samples in TCGA-PAAD). Load the Bailey ADEX, Bailey Immunogenic, and Collisson Exocrine signatures from the pdacR gene_lists object, then z-score the signature genes per sample and compute mean-centered gene set scores (mean of z-scored signature genes) for each matched sample class. The pairing structure is what licenses the comparison: score drops are evaluated within patient pairs, not across cohort averages.

## Related tools

- **pdacR** (Supplies the parsed PACA-AU RNAseq/microarray expression matrices, sampInfo metadata for paired-sample identification, ICGC_PARSE_Nones filter flags, and the gene_lists object with the ADEX/Immunogenic/Exocrine signatures) — https://github.com/rmoffitt/pdacR
- **R** (Runtime environment in which the pdacR package objects (.rds) are loaded, samples are paired via sampInfo, and z-scored signature scores are computed) — https://github.com/rmoffitt/pdacR

## Examples

```
devtools::install_github("rmoffitt/pdacR"); pdacR::pdacShiny()
```

## Evaluation signals

- Every sample in the output table carries a class label (primary/metastatic/cell line) and a platform (RNAseq or microarray); no ICGC_PARSE_Nones-flagged samples remain
- Each metastatic or cell-line sample is linkable to a matched primary tumor via sampInfo within or across the two PACA-AU platforms
- Each of the three signatures (Bailey ADEX, Bailey Immunogenic, Collisson Exocrine) shows a clear reduction in mean-centered score in derived cell lines and metastases versus paired primary tumors on both PACA-AU platforms (Fig. 2c expectation)
- The score table written to icgc_gene_set_scores.csv has shape samples x 3 signatures and scores are mean-centered per sample (mean of z-scored signature genes)

## Limitations

- Cross-platform matching is fragile: 'the inability to robustly merge datasets' and the fact that datasets 'come from a variety of platforms, including bulk RNA sequencing and microarrays' add complexity to aggregation and batch correction
- pdacR removed data aggregation 'because it introduced more problems than it solved', so pairs must be analyzed per-platform rather than pooled naively
- Bulk transcriptomic data limit the ability to fully separate cellular populations, so signature score drops in cell lines may partly reflect absent stromal/immune components rather than tumor-intrinsic subtype switching

## Evidence

- [readme] Data/ holds all parsed datasets, as well as the gene signatures object and PurIST classifier object.: "Data/ holds all parsed datasets, as well as the gene signatures object and PurIST classifier object. These datasets are used both in the app and in the associated publication"
- [readme] Sample filtering via label checkboxes: "Checking a box in this list will remove all samples with that label from all analyses. This is important to do to ensure accruate analyses."
- [readme] TCGA filtering example: [evidence span withheld — anchor status: not_found]
- [discussion] Multi-platform complexity limitation: "our datasets come from a variety of platforms, including bulk RNA sequencing and microarrays, adding another layer of complexity to aggregation and batch correction"
- [discussion] pdacR houses annotated PDAC data: "we developed an R package termed pdacR that houses carefully organized and annotated public PDAC transcriptomics data for ease of access and heightened transparency"
