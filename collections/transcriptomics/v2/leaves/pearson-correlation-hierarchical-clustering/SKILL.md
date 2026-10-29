---
name: pearson-correlation-hierarchical-clustering
description: Use when you have deep features from a trained DeepCC model and need
  to test whether features self-organize into biological groups (e.g., three groups
  mapping to CMS1 MSI, CMS3 metabolic, CMS4 EMT).
when_to_use_negative:
- Input is raw gene expression or functional spectra rather than deep features extracted
  from a trained DeepCC ANN — the grouping structure exists in learned-feature space,
  not in the raw pathway-activity table
- Goal is classifying new patient samples rather than interpreting model internals
  (use get_DeepCC_label for prediction; DeepCC is a classification framework, not
  for clustering of cancer patients)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DeepCC
  role: Provides get_DeepCC_features to extract deep features from the last hidden
    layer of the trained ANN and vis_samples to render sample heatmaps of the feature
    groups
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: R
  role: Implementation environment for the correlation matrix computation and hierarchical
    clustering of the deep features
- name: ggplot2
  role: Used with cowplot (theme_cowplot) to render the Fig. 3b-d style heatmaps via
    vis_samples
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:ef9805fd6ada2a971870c0db4c227533fba1b4f7796f093fd36676bc1cf1a93b
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pearson-correlation-hierarchical-clustering

## Summary

An interpretability workflow from the DeepCC paper: compute the absolute Pearson correlation matrix among the ten deep features extracted from the last hidden layer of a trained DeepCC ANN, hierarchically cluster that matrix, and annotate the resulting feature groups against subtype-relevant biology (MSI, metabolic dysregulation, EMT) to show the network learned CMS-subtype characteristics.

## When to use

Use when you have deep features from a trained DeepCC model and need to test whether features self-organize into biological groups (e.g., three groups mapping to CMS1 MSI, CMS3 metabolic, CMS4 EMT).

## When NOT to use

- Input is raw gene expression or functional spectra rather than deep features extracted from a trained DeepCC ANN — the grouping structure exists in learned-feature space, not in the raw pathway-activity table
- Goal is classifying new patient samples rather than interpreting model internals (use get_DeepCC_label for prediction; DeepCC is a classification framework, not for clustering of cancer patients)

## Inputs

- Deep feature matrix (10 features per sample) from the last hidden layer of the CRC_TCGA DeepCC model, obtained via get_DeepCC_features(deepcc_model, fs)
- Functional spectra (fs) computed with getFunctionalSpectra on gene expression profiles using MSigDB v7 (22,596 gene sets)
- CMS subtype labels (e.g., CMS1/CMS2/CMS3/CMS4) for the samples

## Outputs

- 10x10 absolute Pearson correlation matrix among deep features
- Hierarchical clustering of features yielding three feature groups
- Group annotations mapping feature groups to MSI, metabolic dysregulation, and higher EMT (Fig. 3b-d style heatmaps, Table S5)

## How to apply

First extract deep features from the last hidden layer of the trained CRC_TCGA DeepCC model with get_DeepCC_features(deepcc_model, fs), where fs are functional spectra computed by getFunctionalSpectra on the MSigDB v7 gene sets (22,596 gene sets, default). Compute the absolute Pearson correlation matrix among the ten deep features; taking absolute values lets both positively and negatively co-active features cluster together. Hierarchically cluster the correlation matrix and cut to three groups, since the CMS subtypes (CMS1/CMS3/CMS4) are each expected to be captured by one feature group. Annotate each cluster against MSI status, metabolic dysregulation, and EMT signatures and render sample-level heatmaps with vis_samples(df, labels, color) from DeepCC (ggplot2/cowplot). Validate acceptance by checking that exactly three feature groups emerge and map to MSI, metabolic, and EMT, and optionally confirm the deep-feature space is more compact (average silhouette widths per CMS label) than the 273-gene CMS signature space via a Wilcoxon signed-rank test at P < 0.01, matching Fig. 3b-d and Table S5.

## Related tools

- **DeepCC** (Provides get_DeepCC_features to extract deep features from the last hidden layer of the trained ANN and vis_samples to render sample heatmaps of the feature groups) — https://github.com/CityUHK-CompBio/DeepCC
- **R** (Implementation environment for the correlation matrix computation and hierarchical clustering of the deep features)
- **ggplot2** (Used with cowplot (theme_cowplot) to render the Fig. 3b-d style heatmaps via vis_samples)

## Examples

```
cor_mat <- abs(cor(df)); hc <- hclust(as.dist(1 - cor_mat)); groups <- cutree(hc, k = 3)
```

## Evaluation signals

- Hierarchical clustering of the absolute Pearson correlation matrix resolves exactly three deep-feature groups, not two or four
- The three groups annotate cleanly onto MSI (CMS1), metabolic dysregulation (CMS3), and higher EMT (CMS4), reproducing Fig. 3b-d and Table S5
- All entries of the correlation matrix lie in [0, 1] because absolute values are used
- Deep-feature average silhouette widths grouped by CMS labels significantly exceed 273-gene signature silhouettes in a Wilcoxon signed-rank test at P < 0.01

## Limitations

- DeepCC, and hence this interpretability analysis, is a classification framework and not applicable to clustering of cancer patients
- Reliable grouping depends on comprehensive gene set prior knowledge — major characteristic signaling pathways for each subtype must be included in the gene set database (MSigDB)
- The three-group MSI/metabolic/EMT interpretation is specific to the CRC CMS subtypes and the ten deep features of the CRC_TCGA model; other cancers may yield different group structures

## Evidence

- [results] Clustering of absolute Pearson correlation coefficients between the ten deep features yielded three distinct feature groups: [evidence span withheld — anchor status: not_found]
- [methods] Compute the absolute Pearson correlation matrix among the deep features and hierarchically cluster it to identify three feature groups.: [evidence span withheld — anchor status: not_found]
- [abstract] deep features learned by DeepCC captured biological characteristics associated with distinct molecular subtypes, enabling more compact within-subtype distribution and between-subtype separation: "deep features learned by DeepCC captured biological characteristics associated with distinct molecular subtypes, enabling more compact within-subtype distribution and between-subtype separation"
- [readme] By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra.: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra."
