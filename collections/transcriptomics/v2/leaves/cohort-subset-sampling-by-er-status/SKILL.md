---
name: cohort-subset-sampling-by-er-status
description: Use when testing sensitivity of a nearest-centroid subtype classifier
  (PAM50) to the composition of the cohort used for reference construction, requiring
  ER-stratified random subsets.
when_to_use_negative:
- The goal is a single-sample, reference-free classifier (e.g., mLR or AIMS); composition-controlled
  reference construction is irrelevant there
- Input cohort lacks reliable ER status annotations, since ER-stratified 50/50 or
  60/40 sampling cannot be enforced
- The classifier is not reference-based (no cohort medoids/centroids to recompute),
  so subset composition cannot affect its calls
edam_topics:
- http://edamontology.org/topic_3379
tools:
- name: PAM50
  role: Nearest-centroid subtype classifier whose medoid-based reference is rebuilt
    on each ER-stratified subset
- name: AIMS (Absolute Intrinsic Molecular Subtyping)
  role: Alternative single-sample subtyping approach referenced as context for reference-free
    classification
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:079774a68af36391e868d144e5d62779ab125850e51072278b57232d8bc0afba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AIMS (Absolute Intrinsic Molecular Subtyping)
  - PAM50
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cohort-subset-sampling-by-er-status

## Summary

Draws random subsets of a breast cancer RNA-seq cohort with controlled size and ER+/ER− composition (e.g., 262 samples at 50/50) to build PAM50 references and quantify how reference-cohort composition drives subtype-call concordance.

## When to use

Use when testing sensitivity of a nearest-centroid subtype classifier (PAM50) to the composition of the cohort used for reference construction, requiring ER-stratified random subsets.

## When NOT to use

- The goal is a single-sample, reference-free classifier (e.g., mLR or AIMS); composition-controlled reference construction is irrelevant there
- Input cohort lacks reliable ER status annotations, since ER-stratified 50/50 or 60/40 sampling cannot be enforced
- The classifier is not reference-based (no cohort medoids/centroids to recompute), so subset composition cannot affect its calls

## Inputs

- TCGA breast cancer RNA-seq expression matrix (pandas DataFrame)
- Published PAM50 subtype labels per sample
- ER status annotations (ER+/ER−) per sample

## Outputs

- Twenty ER-stratified random sample subsets (ten 60/40 of 400 samples, ten 50/50 of 262 samples)
- Per-reference PAM50 subtype calls against each of the 21 references
- concordance_results.csv with per-reference and group-averaged concordance values

## How to apply

Load the TCGA breast cancer RNA-seq expression matrix with published PAM50 labels and ER status in pandas, then draw multiple random subsets at matched dimensionality and ER proportion — in the source study, ten subsets of 400 samples (60% ER+/40% ER−) and ten subsets of 262 samples (50% ER+/50% ER−) mirroring the Ciriello et al. technical control. For each subset, build a PAM50 reference via medoid-based centroid recomputation on the 50-gene panel, classify all TCGA samples against each reference using the PAM50 nearest-centroid rule (Spearman correlation to subtype medians), and average concordance with published labels per reference group. Rationale: standard PAM50 is strongly affected by reference-cohort composition, so ER-balanced subset sampling isolates the composition effect; matching the original control's composition (262, 50/50) is expected to raise concordance (reported: 85.52% for 60/40 vs 95.45% for 50/50 vs 99.27% for the Ciriello control).

## Related tools

- **PAM50** (Nearest-centroid subtype classifier whose medoid-based reference is rebuilt on each ER-stratified subset)
- **AIMS (Absolute Intrinsic Molecular Subtyping)** (Alternative single-sample subtyping approach referenced as context for reference-free classification)

## Examples

```
ref = pd.concat([df[df.ER=='pos'].sample(131), df[df.ER=='neg'].sample(131)])
```

## Evaluation signals

- Mean concordance of the ten 60/40-400-sample references must be ~85.52%
- Mean concordance of the ten 50/50-262-sample references must be ~95.45%
- Concordance of the Ciriello et al. technical control reference must be ~99.27%
- Each drawn subset must exactly match its specified size (400 or 262) and ER proportion (60/40 or 50/50)
- All per-reference concordance values must be written to concordance_results.csv

## Limitations

- Published subtype labels were themselves obtained with the PAM50 assay on the same gene panel, biasing concordance estimates toward PAM50-based references
- The Normal-like class has undetermined clinical significance and degrades classification performance
- Ambiguous samples, poorly or comparably correlated to more than one class centroid, reduce classification stability and concordance

## Evidence

- [abstract] Standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction.: "standard PAM50 is profoundly affected by the composition of the sample cohort used for reference construction"
- [discussion] Dataset composition affects subtype calling: "dataset composition and choices for reference construction affect subsequent subtype calling"
- [readme] README on AWCA versions of PAM50 for BC intrinsic subtyping: "The R-code we provide allows to use our _AWCA_ versions of PAM50 method"
