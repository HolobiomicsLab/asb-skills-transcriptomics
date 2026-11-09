---
name: subtyping-comparison-visualization
description: Use when you need to cross-compare intrinsic breast cancer subtype callers
  (Prosigna, standard PAM50, AWCA-based PAM50) on the same RNA-seq profiles and visualize
  call concordance plus ROR prognostic agreement.
when_to_use_negative:
- Input is a single-sample classification task rather than a cross-caller comparison
  — use AWCA_PAM50_BC_subtyping or mLR directly
- Expression data are microarray-based (e.g. Affymetrix GSE4922/GSE1456) and the goal
  is mLR signature validation rather than Prosigna-vs-PAM50 ROR comparison
- No survival endpoints are available, so the 10-year overall survival discrimination
  cannot be evaluated
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Prosigna
  role: Nearest-shrunken-centroid classifier used to assign subtypes (Normal-like
    excluded) and compute ROR scores in silico on TCGA RNA-seq profiles
  repo: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
- name: AWCA_PAM50_BC_subtyping
  role: Provides the AWCA-based PAM50 ROR-C calls and the precomputed external reference
    used for comparison
  repo: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:c2ea06c3bba83d746166eef0c95415c0e072f9f37e32938cfb8113acb25e2a97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
  tools:
  - AWCA_PAM50_BC_subtyping
  - Prosigna
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# subtyping-comparison-visualization

## Summary

Reproduce in-silico Prosigna subtyping and ROR scoring on TCGA breast cancer RNA-seq PAM50 profiles, compare Prosigna calls/ROR against standard and AWCA-based PAM50 ROR-C calls for 10-year overall survival discrimination, and render the agreement as Figure 3 (figure3_prosigna_vs_pam50.png).

## When to use

Use when you need to cross-compare intrinsic breast cancer subtype callers (Prosigna, standard PAM50, AWCA-based PAM50) on the same RNA-seq profiles and visualize call concordance plus ROR prognostic agreement.

## When NOT to use

- Input is a single-sample classification task rather than a cross-caller comparison — use AWCA_PAM50_BC_subtyping or mLR directly
- Expression data are microarray-based (e.g. Affymetrix GSE4922/GSE1456) and the goal is mLR signature validation rather than Prosigna-vs-PAM50 ROR comparison
- No survival endpoints are available, so the 10-year overall survival discrimination cannot be evaluated

## Inputs

- TCGA breast cancer RNA-seq expression profiles (PAM50 gene panel)
- Precomputed AWCA external reference (Supplementary Material Table S3)
- 10-year overall survival labels (good vs poor) for prognostic discrimination

## Outputs

- Prosigna subtype calls per sample (Normal-like class excluded)
- Prosigna ROR scores, standard PAM50 ROR-C scores, AWCA-based PAM50 ROR-C scores
- Concordance table of Prosigna vs standard/AWCA PAM50 calls
- Figure 3 rendered as figure3_prosigna_vs_pam50.png

## How to apply

Load TCGA breast cancer RNA-seq expression restricted to the PAM50 gene panel and normalize the profiles with the Prosigna algorithm, but substitute the precomputed AWCA external reference (Supplementary Material Table S3) for a cohort-derived reference. Classify each sample with the Prosigna nearest-shrunken-centroid classifier excluding the Normal-like class, and compute the Prosigna Risk Of Recurrence (ROR) score from the ROR model. On the same profiles compute the standard PAM50 and AWCA-based PAM50 ROR-C scores, then compare Prosigna subtype calls against the PAM50-based calls via call concordance and ROR score agreement. Finally render the three-way comparison as Figure 3 and save it as figure3_prosigna_vs_pam50.png; the rationale is that standard PAM50 is biased by reference-cohort composition, whereas AWCA uses a fixed external reference so concordance among AWCA-based calls exceeds 98%.

## Related tools

- **Prosigna** (Nearest-shrunken-centroid classifier used to assign subtypes (Normal-like excluded) and compute ROR scores in silico on TCGA RNA-seq profiles) — https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
- **AWCA_PAM50_BC_subtyping** (Provides the AWCA-based PAM50 ROR-C calls and the precomputed external reference used for comparison) — https://github.com/DEIB-GECO/BC_Intrinsic_subtyping

## Examples

```
ggsave("figure3_prosigna_vs_pam50.png", fig3_prosigna_vs_pam50, width = 8, height = 6)
```

## Evaluation signals

- Rendered figure figure3_prosigna_vs_pam50.png exists on disk
- Prosigna-vs-AWCA call concordance is consistent with reported robustness: AWCA-based concordance beyond 98%
- Standard PAM50 vs AWCA-based PAM50 concordance is above 90%
- Standard and AWCA-based PAM50 ROR scores are highly correlated across samples
- AWCA-based ROR-C reaches the most statistically significant p-value in discriminating good vs poor 10-year overall survival; Prosigna shows a slightly more pessimistic trend

## Limitations

- Standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction, limiting robustness and reproducibility of subtype calling
- Prosigna normalization requires substituting the precomputed AWCA external reference; a cohort-derived reference reintroduces the reference-composition bias
- The Normal-like class has undetermined clinical significance and degrades performance, which is why it is excluded from Prosigna calls here

## Evidence

- [discussion] AWCA external reference instead of cohort-derived reference: "the AWCA strategy allows constructing predefined external references (Supplementary Material Table S3) that can be used to classify independent RNA-seq profiles"
- [discussion] AWCA concordance robustness threshold: "The concordance achieved by AWCA-based classifications among them is beyond 98%"
- [abstract] reference-cohort bias motivating comparison: "standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction"
- [readme] classifier scope (five subtypes incl. Normal-like in general usage): "to assign breast cancer (BC) samples to the five prognosis-related breast cancer _intrinsic_ subtypes (_Luminal A, Luminal B, Basal, Her2-enriched, or Normal-like_)"
