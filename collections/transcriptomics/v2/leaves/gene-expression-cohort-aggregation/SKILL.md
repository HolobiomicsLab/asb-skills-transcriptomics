---
name: gene-expression-cohort-aggregation
description: Use when building a subtype reference for PAM50-style classification
  from multiple subtype-labeled RNA-seq cohorts and the goal is a cohort-composition-independent
  centroid table (e.g. matching Supplementary Material Table S3).
when_to_use_negative:
- The task is single-sample classification without reference construction — use the
  authors' mLR single-sample caller instead, which reaches 90% concordance with PAM50
  without needing a reference sample
- The input data lacks subtype labels for the training cohorts — the within-class
  averages require existing PAM50 (or preliminary) subtype assignments per sample
- The goal is to re-derive the standard cohort-internal PAM50 reference for the dataset
  under study rather than a fixed external reference
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: AWCA_PAM50_BC_subtyping
  role: Downstream R-code that uses the AWCA reference table to assign breast cancer
    samples to the five intrinsic subtypes
  repo: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping/wiki/AWCA_PAM50_BC_subtyping
- name: BC_Intrinsic_subtyping
  role: Authors' repository providing the PAM50-labeled training cohorts and R-code
    for AWCA-PAM50 classification
  repo: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:4926bf5971cf612a2950f1fe6edce7f30bd5f4f800f453cbe73de7f9bf7503ad
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AWCA_PAM50_BC_subtyping
  - BC_Intrinsic_subtyping
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gene-expression-cohort-aggregation

## Summary

Constructs an AWCA (average of within-class averages) reference table for PAM50 breast cancer subtyping by aggregating per-gene, per-subtype mean expression across training cohorts, removing the dependence of subtype calls on the arbitrary composition of the reference cohort.

## When to use

Use when building a subtype reference for PAM50-style classification from multiple subtype-labeled RNA-seq cohorts and the goal is a cohort-composition-independent centroid table (e.g. matching Supplementary Material Table S3).

## When NOT to use

- The task is single-sample classification without reference construction — use the authors' mLR single-sample caller instead, which reaches 90% concordance with PAM50 without needing a reference sample
- The input data lacks subtype labels for the training cohorts — the within-class averages require existing PAM50 (or preliminary) subtype assignments per sample
- The goal is to re-derive the standard cohort-internal PAM50 reference for the dataset under study rather than a fixed external reference

## Inputs

- PAM50-labeled gene expression training matrices (RNA-seq or log2-transformed Affymetrix data, e.g. GEO datasets GSE4922 and GSE1456) with subtype labels per sample
- List of the five intrinsic subtype labels (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like)

## Outputs

- awca_reference_table.csv — a gene-by-subtype centroid table structurally matching the authors' precomputed references (Supplementary Material Table S3)

## How to apply

Load each subtype-labeled gene expression training cohort (e.g. PAM50-labeled matrices from github.com/DEIB-GECO/BC_Intrinsic_subtyping) with pandas. For each cohort, compute per-gene mean expression within each of the five intrinsic subtypes (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like) via a groupby aggregation over subtype labels — this is the 'within-class average'. Average these within-subtype class means across all training cohorts per gene to obtain the AWCA reference vector, which is independent of subtype numerosity and mitigates cohort-composition bias of the standard PAM50 reference. Assemble the resulting gene-by-subtype centroid table formatted to match the authors' precomputed references (Supplementary Material Table S3) and save as awca_reference_table.csv. Decision point: verify the table structurally matches the precomputed references (same gene and subtype columns) before using it for downstream AWCA-PAM50 classification, which should reach beyond-98% concordance among AWCA-based classifications.

## Related tools

- **AWCA_PAM50_BC_subtyping** (Downstream R-code that uses the AWCA reference table to assign breast cancer samples to the five intrinsic subtypes) — https://github.com/DEIB-GECO/BC_Intrinsic_subtyping/wiki/AWCA_PAM50_BC_subtyping
- **BC_Intrinsic_subtyping** (Authors' repository providing the PAM50-labeled training cohorts and R-code for AWCA-PAM50 classification) — https://github.com/DEIB-GECO/BC_Intrinsic_subtyping

## Examples

```
import pandas as pd; df = pd.read_csv('cohort.csv'); wca = df.groupby('subtype').mean(); awca = wca.groupby(level=1).mean(); awca.T.to_csv('awca_reference_table.csv')
```

## Evaluation signals

- Structural check: awca_reference_table.csv has the same gene and subtype columns as the authors' precomputed references in Supplementary Material Table S3
- All five subtypes (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like) are represented as columns in the centroid table
- Downstream AWCA-based classifications built from the table achieve beyond-98% concordance among AWCA-based classifications
- AWCA-based classifications show over 90% concordance with original calls and retain prognostic ability on independent datasets (e.g. GSE4922, GSE1456)
- Per-gene reference values are averaged across cohorts, confirming the result is independent of subtype numerosity in any single cohort

## Limitations

- Standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction; naive single-cohort references reproduce this bias
- Performance of labels trained on PAM50-derived calls is biased because published subtype labels were themselves obtained with the PAM50 assay on the same gene panel
- The Normal-like class, whose clinical significance remains undetermined, causes degradation of classification performance
- A small number of training samples and a vast number of features limit subtyping capability, mainly for ambiguous samples poorly or comparably correlated to more than one class centroid

## Evidence

- [abstract] Standard PAM50 reference-cohort bias mitigated: "standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction"
- [abstract] AWCA improves robustness: "we propose a strategy, named AWCA, to mitigate this issue, improving classification robustness, with over 90% of concordance, and prognostic ability"
- [discussion] AWCA yields predefined external references: "the AWCA strategy allows constructing predefined external references (Supplementary Material Table S3) that can be used to classify independent RNA-seq profiles"
- [discussion] AWCA-based classifications inter-concordance: "The concordance achieved by AWCA-based classifications among them is beyond 98%"
- [readme] Repository provides AWCA-PAM50 subtyping code: [evidence span withheld — anchor status: not_found]
