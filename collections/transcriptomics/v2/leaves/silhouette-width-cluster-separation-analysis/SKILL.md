---
name: silhouette-width-cluster-separation-analysis
description: Use when you need to validate that a learned feature space (e.g. ANN
  deep features) yields more compact within-group and better separated between-group
  clusters than a baseline feature space on the same labeled samples.
when_to_use_negative:
- When the goal is unsupervised discovery of subtypes rather than validating compactness
  of existing labels — DeepCC is specifically developed for classification, not clustering
  of cancer patients
- When samples lack subtype labels, since silhouette widths must be grouped by known
  CMS (or equivalent) labels
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: DeepCC
  role: Provides get_DeepCC_features to extract last-hidden-layer deep features and
    vis_samples for heatmap visualization of samples
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: keras
  role: Loads the trained CRC DeepCC model via keras::load_model_hdf5 for deep-feature
    extraction
- name: ggplot2
  role: Plotting backend for vis_samples heatmaps of samples in deep-feature space
- name: cowplot
  role: theme_cowplot styling used with vis_samples for Fig. 3b-d style plots
- name: R
  role: Implementation environment for silhouette computation, Wilcoxon signed-rank
    test, and hierarchical clustering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:a580e7e2163bc276f578323f8ffa269e3859b95d243fd52c73494fbb03da0549
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
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# Silhouette-width cluster separation analysis

## Summary

Quantifies within-subtype compactness and between-subtype separation of patient samples by average silhouette width in a learned feature space (DeepCC deep features) versus a baseline space (273-gene CMS signature), with paired Wilcoxon signed-rank testing, to demonstrate that deep features capture subtype biology.

## When to use

Use when you need to validate that a learned feature space (e.g. ANN deep features) yields more compact within-group and better separated between-group clusters than a baseline feature space on the same labeled samples.

## When NOT to use

- When the goal is unsupervised discovery of subtypes rather than validating compactness of existing labels — DeepCC is specifically developed for classification, not clustering of cancer patients
- When samples lack subtype labels, since silhouette widths must be grouped by known CMS (or equivalent) labels

## Inputs

- Trained DeepCC model files (CRC_TCGA .hdf5 plus .RData with levels)
- Functional spectra (fs) computed from gene expression profiles with getFunctionalSpectra on MSigDB v7 (22,596 gene sets)
- Sample CMS subtype labels
- 273-gene CMS signature expression values for the same samples

## Outputs

- Average silhouette widths per hidden-layer depth grouped by CMS label
- Paired deep-feature vs 273-gene signature silhouette widths with Wilcoxon signed-rank P value
- Absolute Pearson correlation matrix among the ten deep features and its hierarchical clustering into three feature groups
- Fig. 3b-d style sample heatmaps rendered via vis_samples

## How to apply

Load the trained DeepCC model with load_DeepCC_model('CRC_TCGA') (keras::load_model_hdf5 on the .hdf5 plus .RData levels) and extract the ten deep features from the last hidden layer with get_DeepCC_features(deepcc_model, fs) on functional spectra computed by getFunctionalSpectra (MSigDB v7, 22,596 gene sets). Compute average silhouette widths of samples grouped by their CMS labels at each hidden-layer depth of the trained network to track how compactness/separation evolves with depth. On the same samples, compute average silhouette widths in deep-feature space and in the 273-gene CMS signature space, then test the paired difference with the Wilcoxon signed-rank test at significance threshold P < 0.01. Additionally compute the absolute Pearson correlation matrix among deep features and hierarchically cluster it to identify three feature groups, annotating them against MSI, metabolic dysregulation, and EMT, and render Fig. 3b-d style heatmaps with vis_samples(df, labels, color) (ggplot2/cowplot).

## Related tools

- **DeepCC** (Provides get_DeepCC_features to extract last-hidden-layer deep features and vis_samples for heatmap visualization of samples) — https://github.com/CityUHK-CompBio/DeepCC
- **keras** (Loads the trained CRC DeepCC model via keras::load_model_hdf5 for deep-feature extraction)
- **ggplot2** (Plotting backend for vis_samples heatmaps of samples in deep-feature space)
- **cowplot** (theme_cowplot styling used with vis_samples for Fig. 3b-d style plots)
- **R** (Implementation environment for silhouette computation, Wilcoxon signed-rank test, and hierarchical clustering)

## Examples

```
CRC_TCGA <- load_DeepCC_model("CRC_TCGA"); fs <- getFunctionalSpectra(eps); df <- get_DeepCC_features(CRC_TCGA$classifer, fs); vis_samples(df, labels, color)
```

## Evaluation signals

- Exactly three deep-feature groups emerge from hierarchical clustering of the absolute Pearson correlation matrix, mapping onto MSI, metabolic dysregulation, and EMT
- Deep-feature average silhouette widths significantly exceed 273-gene CMS signature silhouettes on the same samples (Wilcoxon signed-rank P < 0.01)
- Silhouette widths grouped by CMS labels increase across hidden-layer depths, matching Fig. 3b-d and Table S5
- Extracted deep-feature matrix has ten columns (one per last-hidden-layer unit) with one row per sample

## Limitations

- DeepCC is specifically developed for classification, not clustering of cancer patients
- DeepCC depends on comprehensive gene set prior knowledge (e.g. MSigDB v7, 22,596 gene sets) to avoid potential bias
- Deep features are tied to the trained network; the CRC_TCGA model is trained on TCGA-COADREAD data (626 samples), so transfer to other platforms/cancers requires retraining or the appropriate pre-trained model

## Evidence

- [abstract] deep features learned by DeepCC captured biological characteristics associated with distinct molecular subtypes, enabling more compact within-subtype distribution and between-subtype separation of: "deep features learned by DeepCC captured biological characteristics associated with distinct molecular subtypes, enabling more compact within-subtype distribution and between-subtype separation of"
- [methods] obtain deep features: "# obtain deep features 
df <- get_DeepCC_features(deepcc_model, fs)"
- [methods] CRC_TCGA <- load_DeepCC_model("CRC_TCGA"): "CRC_TCGA <- load_DeepCC_model("CRC_TCGA")"
- [readme] CRC model trained with TCGA data: "load_DeepCC_model <- function(prefix){
  load(file = paste0(prefix, ".RData"))
  classifer <- keras::load_model_hdf5(filepath =paste0(prefix, ".hdf5"))
  list(classifier = classifer, levels ="
- [methods] By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra.: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra."
