---
name: cross-cohort-subtype-classification
description: Use when independent breast cancer RNA-seq cohorts (e.g., GSE96058, PanCA,
  GSE81538, TCGA) must be assigned PAM50 intrinsic subtypes using a reference computed
  on a different cohort, and subtype-call concordance must be quantified.
when_to_use_negative:
- Input is microarray data (e.g., GSE4922, GSE1456 Affymetrix) rather than RNA-seq;
  use the AWCA-based classification route validated on Affymetrix log2-transformed
  data instead of the RNA-seq cross-cohort porting scheme
- Only PAM50 gene-panel data (e.g., Nanostring nCounter/Prosigna) is available rather
  than global RNA-seq expression; use the platform-native assay
- A reference-free single-sample caller is preferred and no pre-defined external reference
  exists; consider the regularized multiclass logistic regression (mLR) models, which
  reached 90% concordance with PAM50 calls without need of a reference sample
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0625
tools:
- name: limma
  role: limma-derived gene selection (e.g., limma50_BWE backward-elimination signature)
    supplies the feature space used in the subtyping framework whose AWCA references
    are ported across cohorts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:fd9b5783e7f4e7c467b15cda709cfa200f633b8fa585bff22bcd24d5c3475030
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - limma
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cross-cohort-subtype-classification

## Summary

Port AWCA-PAM50 breast cancer intrinsic subtyping references built on one RNA-seq cohort to classify independent cohorts (e.g., TCGA RSEM reference to PanCA; GSE96058 FPKM reference to GSE81538), validating against internal-reference calls with concordance thresholds of ~95% internal and >96% external.

## When to use

Use when independent breast cancer RNA-seq cohorts (e.g., GSE96058, PanCA, GSE81538, TCGA) must be assigned PAM50 intrinsic subtypes using a reference computed on a different cohort, and subtype-call concordance must be quantified.

## When NOT to use

- Input is microarray data (e.g., GSE4922, GSE1456 Affymetrix) rather than RNA-seq; use the AWCA-based classification route validated on Affymetrix log2-transformed data instead of the RNA-seq cross-cohort porting scheme
- Only PAM50 gene-panel data (e.g., Nanostring nCounter/Prosigna) is available rather than global RNA-seq expression; use the platform-native assay
- A reference-free single-sample caller is preferred and no pre-defined external reference exists; consider the regularized multiclass logistic regression (mLR) models, which reached 90% concordance with PAM50 calls without need of a reference sample

## Inputs

- Raw/reprocessed RNA-seq expression matrices for GSE96058, PanCA and GSE81538 from the repository data directory
- TCGA RSEM expression profiles for building the external AWCA reference applied to PanCA
- GSE96058 FPKM expression profiles for building the external AWCA reference applied to GSE81538
- Set of genes shared across all cohorts (intersection feature restriction)

## Outputs

- AWCA-PAM50 subtype calls (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like) per sample per cohort, internal and external-reference
- awca_concordance_results.csv: table of internal and external concordance percentages per cohort pair

## How to apply

Load raw/reprocessed RNA-seq expression matrices for GSE96058, PanCA and GSE81538 and restrict features to genes shared across cohorts. Build internal AWCA references per cohort: for every sample, compute Spearman correlations against the remaining samples of that cohort and call the subtype of the nearest-centroid sample following the AWCA single-sample scheme (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like). Then apply external references: the best TCGA AWCA reference (built from TCGA RSEM profiles) to PanCA, and the GSE96058 FPKM reference to GSE81538, as pre-defined external references per Supplementary Material Table S3. Compute pairwise concordance of internal calls within each cohort (expect ~95% for GSE96058, ~95% for PanCA and GSE81538) and of external-reference calls against internal calls (expect beyond 96%), then assemble a per-pair concordance table (awca_concordance_results.csv). Rationale: the AWCA strategy mitigates the reference-cohort bias of standard PAM50, which is profoundly affected by the composition of the reference cohort.

## Related tools

- **limma** (limma-derived gene selection (e.g., limma50_BWE backward-elimination signature) supplies the feature space used in the subtyping framework whose AWCA references are ported across cohorts)

## Evaluation signals

- Internal AWCA concordance for GSE96058 is at least 95%
- External-reference concordances (TCGA reference on PanCA; GSE96058 reference on GSE81538) exceed 96%
- Internal concordances for PanCA and GSE81538 are approximately 95%, matching the paper-reported values
- Feature spaces are restricted to the gene intersection shared across all cohorts before correlation computation
- awca_concordance_results.csv is produced with one concordance percentage per cohort pair (internal and external)

## Limitations

- Standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction; reference choices affect subtype calling and limit robustness and reproducibility
- Ambiguous samples that are poorly or comparably correlated to more than one class centroid reduce classification stability
- Concordance with published labels is partly biased because those labels were themselves obtained with the PAM50 assay on the same gene panel
- The Normal-like class has undetermined clinical significance and causes degradation of performance
- A small number of training samples versus a vast number of features limits subtyping capability for the hardest-to-recognize cases

## Evidence

- [discussion] the AWCA strategy allows constructing predefined external references (Supplementary Material Table S3) that can be used to classify independent RNA-seq profiles: "the AWCA strategy allows constructing predefined external references (Supplementary Material Table S3) that can be used to classify independent RNA-seq profiles"
- [abstract] we propose a strategy, named AWCA, to mitigate this issue, improving classification robustness, with over 90% of concordance: "we propose a strategy, named AWCA, to mitigate this issue, improving classification robustness, with over 90% of concordance, and prognostic ability"
- [discussion] The concordance achieved by AWCA-based classifications among them is beyond 98%: "The concordance achieved by AWCA-based classifications among them is beyond 98%"
- [abstract] standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction: "standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction"
- [readme] The R-code we provide allows to use our _AWCA_ versions of PAM50 method: "The R-code we provide allows to use our _AWCA_ versions of PAM50 method"
