---
name: wilcoxon-signed-rank-hypothesis-testing
description: Use when two paired measurements were taken on the same samples (e.g.,
  silhouette widths in deep-feature space vs. the 273-gene CMS signature space) and
  you must test whether one representation is significantly more compact/separated
  than the other without assuming normality.
when_to_use_negative:
- When the two groups of measurements are not paired (independent samples) — the signed-rank
  test requires paired differences on the same samples
- When the goal is unsupervised clustering of cancer patients — the article states
  DeepCC is 'specifically developed for addressing current challenges in classification
  but not clustering of cancer patients'
- When the input is already a prediction/label vector rather than paired continuous
  per-sample scores — use a classifier performance comparison instead
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Implementation language for computing silhouette widths and running the paired
    Wilcoxon signed-rank test (wilcox.test) on deep-feature vs. CMS-signature silhouettes
- name: DeepCC
  role: Supplies the trained CRC_TCGA model and get_DeepCC_features/getFunctionalSpectra
    functions that generate the 10 deep features whose compactness is tested against
    the 273-gene CMS signature
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: keras
  role: Loads the .hdf5 CRC_TCGA ANN from which last-hidden-layer deep features are
    extracted prior to the paired silhouette comparison
  repo: https://github.com/zero19970/deepcc_model
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:833900876f011e06ba9fc9fa8861916f5c64180a7dd355ab53d2118b39c4cf04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CityUHK-CompBio/DeepCC
  tools:
  - DeepCC
  - R
  - cowplot
  - ggplot2
  - keras
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Wilcoxon signed-rank hypothesis testing

## Summary

Non-parametric paired significance test used in the DeepCC CRC study to verify that per-sample average silhouette widths computed in the 10-dimensional deep-feature space of the trained CRC_TCGA ANN significantly exceed those computed in the 273-gene CMS signature space at P < 0.01.

## When to use

Use when two paired measurements were taken on the same samples (e.g., silhouette widths in deep-feature space vs. the 273-gene CMS signature space) and you must test whether one representation is significantly more compact/separated than the other without assuming normality.

## When NOT to use

- When the two groups of measurements are not paired (independent samples) — the signed-rank test requires paired differences on the same samples
- When the goal is unsupervised clustering of cancer patients — the article states DeepCC is 'specifically developed for addressing current challenges in classification but not clustering of cancer patients'
- When the input is already a prediction/label vector rather than paired continuous per-sample scores — use a classifier performance comparison instead

## Inputs

- Paired per-sample average silhouette widths in deep-feature space (from get_DeepCC_features on CRC_TCGA functional spectra)
- Paired per-sample average silhouette widths in the 273-gene CMS signature space computed on the same samples
- CMS subtype labels for each sample

## Outputs

- Wilcoxon signed-rank test statistic and P value comparing the paired silhouette vectors
- Significance decision at threshold P < 0.01 (deep-feature compactness vs. 273-gene signature)

## How to apply

First extract the 10 deep features from the last hidden layer of the trained CRC DeepCC model with get_DeepCC_features(deepcc_model, fs), where fs are functional spectra computed from MSigDB v7 (22,596 gene sets) by getFunctionalSpectra. Compute average silhouette widths per sample grouped by CMS labels (or at each hidden-layer depth) in both the deep-feature space and the 273-gene CMS signature space on the same samples, producing two paired vectors. Apply the one-sided/two-sided Wilcoxon signed-rank test on the paired differences (R: wilcox.test with paired = TRUE) and declare significance at P < 0.01. The analysis is accepted only if deep-feature silhouettes significantly exceed 273-gene signature silhouettes at P < 0.01, matching Fig. 3b-d and Table S5; the rationale is that deep features should concentrate samples within CMS subtypes more tightly than the gene-signature representation.

## Related tools

- **R** (Implementation language for computing silhouette widths and running the paired Wilcoxon signed-rank test (wilcox.test) on deep-feature vs. CMS-signature silhouettes)
- **DeepCC** (Supplies the trained CRC_TCGA model and get_DeepCC_features/getFunctionalSpectra functions that generate the 10 deep features whose compactness is tested against the 273-gene CMS signature) — https://github.com/CityUHK-CompBio/DeepCC
- **keras** (Loads the .hdf5 CRC_TCGA ANN from which last-hidden-layer deep features are extracted prior to the paired silhouette comparison) — https://github.com/zero19970/deepcc_model

## Examples

```
wilcox.test(sil_deep_features, sil_cms273_signature, paired = TRUE)  # reject H0 when p < 0.01
```

## Evaluation signals

- The test must be applied to paired vectors: the same samples must have silhouette widths measured in both deep-feature space and 273-gene CMS signature space
- Significance threshold is P < 0.01 (article's stated criterion for the paired Wilcoxon comparison)
- Direction of effect: deep-feature silhouettes must significantly exceed 273-gene signature silhouettes
- Result should reproduce the Fig. 3b-d compactness/separation pattern and the agreement with Table S5
- Silhouette widths should be averaged over samples grouped by CMS labels to quantify within-subtype compactness and between-subtype separation

## Limitations

- Requires paired, continuous per-sample measurements; it is not appropriate for independent or categorical comparisons
- The test validates feature-space compactness only for classification settings — DeepCC is explicitly not applicable to clustering of cancer patients
- Conclusion depends on deep features coming from the trained CRC_TCGA model with MSigDB v7 (22,596 gene sets) functional spectra; changing the gene-set version or model changes the pairing basis

## Evidence

- [full_text] Paired silhouette comparison workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Deep features improve sample separation: "deep features learned by DeepCC captured biological characteristics associated with distinct molecular subtypes, enabling more compact within-subtype distribution"
- [methods] Deep feature extraction call: "obtain deep features 
df <- get_DeepCC_features(deepcc_model, fs)"
- [readme] Functional spectra default gene sets: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra."
- [discussion] DeepCC scope: classification not clustering: "specifically developed for addressing current challenges in classification but not clustering of cancer patients"
