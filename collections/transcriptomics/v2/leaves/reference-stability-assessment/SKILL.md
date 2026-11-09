---
name: reference-stability-assessment
description: Use when subtyping TCGA RNA-seq breast cancer samples with PAM50 and
  you need to test whether reference-construction strategy (AWCA vs median centroids)
  affects concordance and reproducibility of subtype calls.
when_to_use_negative:
- The input is already a feature/signature score table rather than a raw gene expression
  matrix needing subtype assignment
- The goal is single-sample classification without reference construction (e.g. the
  paper's mLR or AIMS approaches, which classify without a reference sample)
- The expression data are not RNA-seq PAM50-panel profiles (e.g. Affymetrix microarrays
  such as GSE4922/GSE1456 require log2-transformed variants of the AWCA pipeline)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: AWCA
  role: Strategy for constructing subtype reference centroids as averages of within-class
    averages, mitigating reference-cohort bias
  repo: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
- name: AWCA-PAM50
  role: Nearest-centroid classifier that assigns TCGA RNA-seq samples to the five
    intrinsic subtypes using AWCA-built references
  repo: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:87d0daba443c7af422541c4f366ce5fa7461e453c03b758ea5d8bf68743c52ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
  tools:
  - AWCA
  - AWCA-PAM50
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# reference-stability-assessment

## Summary

Quantifies how stable a nearest-centroid PAM50 reference is to the cohort used to build it, by reconstructing references with the AWCA (average of within-class averages) strategy from repeated random ER+/ER− splits and measuring concordance with published subtype calls and among replicate classifications.

## When to use

Use when subtyping TCGA RNA-seq breast cancer samples with PAM50 and you need to test whether reference-construction strategy (AWCA vs median centroids) affects concordance and reproducibility of subtype calls.

## When NOT to use

- The input is already a feature/signature score table rather than a raw gene expression matrix needing subtype assignment
- The goal is single-sample classification without reference construction (e.g. the paper's mLR or AIMS approaches, which classify without a reference sample)
- The expression data are not RNA-seq PAM50-panel profiles (e.g. Affymetrix microarrays such as GSE4922/GSE1456 require log2-transformed variants of the AWCA pipeline)

## Inputs

- TCGA RNA-seq breast cancer gene expression matrix
- Published standard PAM50 subtype calls for the TCGA cohort
- ER+/ER− status labels for split stratification
- Optional predefined external references (Supplementary Material Table S3)

## Outputs

- awca_concordance.tsv with per-reference concordance vs published PAM50 calls
- Mean ± SD concordance of AWCA classifications vs published calls (expected ~91.17% ±0.87%)
- Pairwise concordance matrix / mean ± SD among the ten AWCA classifications (expected ~99.13% ±0.43%)
- Concordance trend across training subset sizes from 400 down to 100

## How to apply

Load the TCGA RNA-seq expression matrix and published standard PAM50 subtype calls from github.com/DEIB-GECO/BC_Intrinsic_subtyping. Draw ten random 60/40 ER+/ER− training/test splits; for each, run the AWCA pipeline on the 60% training subset to construct an external subtype reference from its standard PAM50 labels, mirroring the predefined references of Supplementary Material Table S3. Classify the full TCGA cohort with each AWCA reference via AWCA-PAM50 nearest-centroid assignment, then compute concordance against the published calls (target: 91.17% ±0.87%) and the pairwise concordance among the ten classifications (target: 99.13% ±0.43%). Repeat with training subset sizes shrinking from 400 to 100 samples to verify the reported improvement trend, and write all concordance statistics to awca_concordance.tsv.

## Related tools

- **AWCA** (Strategy for constructing subtype reference centroids as averages of within-class averages, mitigating reference-cohort bias) — https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
- **AWCA-PAM50** (Nearest-centroid classifier that assigns TCGA RNA-seq samples to the five intrinsic subtypes using AWCA-built references) — https://github.com/DEIB-GECO/BC_Intrinsic_subtyping

## Examples

```
R: conc <- sapply(refs, function(r) mean(awca_calls[[r]] == published_calls)); write.table(conc, "awca_concordance.tsv", sep="\t")
```

## Evaluation signals

- Mean concordance of the ten AWCA classifications with published PAM50 calls falls within 91.17% ±0.87%
- Pairwise concordance among the ten AWCA classifications falls within 99.13% ±0.43%
- Concordance improves monotonically (or near-monotonically) as training subset size shrinks from 400 to 100, matching the reported trend
- awca_concordance.tsv exists and contains ten per-reference concordance values plus aggregate mean ± SD
- Classified samples are assigned to exactly one of the five intrinsic subtypes (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like)

## Limitations

- Standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction, limiting robustness and reproducibility of the classification
- Ambiguous samples that are poorly or comparably correlated to more than one class centroid reduce classification stability
- Concordance estimates are biased by the nature of the published subtype calls, which were themselves obtained from the PAM50 assay
- Degradation of performance is partly caused by the Normal-like class, whose clinical significance remains undetermined

## Evidence

- [abstract] AWCA mitigates PAM50 reference-cohort bias: "we propose a strategy, named AWCA, to mitigate this issue, improving classification robustness, with over 90% of concordance"
- [discussion] AWCA enables predefined external references: "the AWCA strategy allows constructing predefined external references (Supplementary Material Table S3) that can be used to classify independent RNA-seq profiles"
- [discussion] Inter-AWCA classification stability: "The concordance achieved by AWCA-based classifications among them is beyond 98%"
- [discussion] Reference-cohort bias limitation: "dataset composition and choices for reference construction affect subsequent subtype calling"
- [discussion] Ambiguous sample instability: "the stability of some ambiguous samples, i.e., poorly or comparably correlated to more than one class centroid"
- [readme] README: AWCA-PAM50 tool availability: "use our _AWCA_ versions of PAM50 method ([AWCA_PAM50_BC_subtyping]"
