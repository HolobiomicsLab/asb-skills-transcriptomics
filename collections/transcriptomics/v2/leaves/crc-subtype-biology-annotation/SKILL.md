---
name: crc-subtype-biology-annotation
description: Use when you have extracted deep features from the last hidden layer
  of a trained CRC DeepCC model and need to interpret which biological characteristics
  (MSI, metabolic, EMT) each feature group encodes relative to CMS subtypes.
when_to_use_negative:
- Goal is de novo discovery/clustering of cancer patients rather than interpreting
  features of a pre-trained classifier — the article states DeepCC is 'specifically
  developed for addressing current challenges in classification but not clustering
  of cancer patients'.
- No trained DeepCC model or functional spectra are available; this annotation step
  requires the last-hidden-layer deep features from a trained network, not raw gene
  expression.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DeepCC
  role: R package providing get_DeepCC_features to extract last-hidden-layer deep
    features and vis_samples to render sample heatmaps
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: keras
  role: Loads the trained CRC_TCGA .hdf5 network so the last hidden layer's deep features
    can be obtained
  repo: https://keras.rstudio.com/
- name: ggplot2
  role: Used with cowplot to render the correlation-derived feature group heatmaps
    (Fig. 3b-d style)
- name: cowplot
  role: Provides theme_cowplot for the vis_samples heatmaps
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:a15ee56d937b1e1d8404802aecc4edcd288789ed4dff65d6ddfd4d35158767a4
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# crc-subtype-biology-annotation

## Summary

Annotates the ten deep features extracted from a trained DeepCC ANN by hierarchically clustering their absolute Pearson correlation matrix into three feature groups and mapping them onto CRC subtype biology (MSI, metabolic dysregulation, EMT), demonstrating that deep features capture CMS1/CMS3/CMS4 characteristics.

## When to use

Use when you have extracted deep features from the last hidden layer of a trained CRC DeepCC model and need to interpret which biological characteristics (MSI, metabolic, EMT) each feature group encodes relative to CMS subtypes.

## When NOT to use

- Goal is de novo discovery/clustering of cancer patients rather than interpreting features of a pre-trained classifier — the article states DeepCC is 'specifically developed for addressing current challenges in classification but not clustering of cancer patients'.
- No trained DeepCC model or functional spectra are available; this annotation step requires the last-hidden-layer deep features from a trained network, not raw gene expression.

## Inputs

- Trained CRC DeepCC model files (.hdf5 plus .RData with label levels), loaded via load_DeepCC_model('CRC_TCGA')
- Functional spectra (fs) computed from gene expression profiles with getFunctionalSpectra using MSigDB v7 (22,596 gene sets)
- CMS subtype labels for the samples (for silhouette-based validation and heatmap annotation)

## Outputs

- Deep-feature data.frame (df) with ten deep features per sample from the last hidden layer
- Absolute Pearson correlation matrix among the ten deep features and its hierarchical clustering into three feature groups
- Feature-group annotations mapping clusters to MSI, metabolic dysregulation, and EMT biology
- Fig. 3b-d style heatmaps rendered with vis_samples (ggplot2/cowplot)
- Paired silhouette-width comparison (deep features vs. 273-gene CMS signature) with Wilcoxon signed-rank P value

## How to apply

Load the trained CRC_TCGA model with load_DeepCC_model('CRC_TCGA') and extract the ten deep features from the last hidden layer via get_DeepCC_features on functional spectra computed with getFunctionalSpectra (MSigDB v7, 22,596 gene sets, default). Compute the absolute Pearson correlation matrix among the ten deep features and hierarchically cluster it to identify exactly three feature groups. Annotate each correlation-derived group against subtype-relevant biology — MSI (CMS1), metabolic dysregulation (CMS3), and EMT (CMS4) — and render Fig. 3b-d style heatmaps with vis_samples(df, labels, color) from DeepCC using ggplot2/cowplot. To validate the feature space, compute average silhouette widths of samples grouped by CMS labels in deep-feature space and in the 273-gene CMS signature space, and test the paired difference with a Wilcoxon signed-rank test at significance threshold P < 0.01; deep-feature silhouettes should significantly exceed signature silhouettes.

## Related tools

- **DeepCC** (R package providing get_DeepCC_features to extract last-hidden-layer deep features and vis_samples to render sample heatmaps) — https://github.com/CityUHK-CompBio/DeepCC
- **keras** (Loads the trained CRC_TCGA .hdf5 network so the last hidden layer's deep features can be obtained) — https://keras.rstudio.com/
- **ggplot2** (Used with cowplot to render the correlation-derived feature group heatmaps (Fig. 3b-d style))
- **cowplot** (Provides theme_cowplot for the vis_samples heatmaps)

## Examples

```
CRC_TCGA <- load_DeepCC_model("CRC_TCGA"); df <- get_DeepCC_features(CRC_TCGA$classifer, fs)
```

## Evaluation signals

- Hierarchical clustering of the absolute Pearson correlation matrix among the ten deep features yields exactly three feature groups
- The three groups map onto MSI (CMS1), metabolic dysregulation (CMS3), and EMT (CMS4), reproducing Fig. 3b-d and Table S5
- Average silhouette widths of CMS-labeled samples in deep-feature space significantly exceed those in 273-gene CMS signature space (Wilcoxon signed-rank test, P < 0.01)
- Deep-feature data.frame contains exactly ten columns (one per last-hidden-layer unit) for all samples
- Functional spectra are computed with the default MSigDB v7 setting (22,596 gene sets) so the feature extraction matches the trained model

## Limitations

- DeepCC is a supervised classification framework, not a clustering tool, so this annotation presupposes a pre-trained subtype classifier and labeled reference subtypes (CMS).
- The prior knowledge of gene sets needs to be comprehensive to avoid potential bias; MSigDB version choice (v5.0: 10,348; v7.0: 17,779 gene sets via alternative settings; default v7: 22,596) changes the functional spectra and thus the deep features.
- Interpretation depends on subtype-relevant biology (MSI, metabolic dysregulation, EMT) actually being represented in the gene set database.

## Evidence

- [abstract] deep features learned by DeepCC captured biological characteristics associated with distinct molecular subtypes: "deep features learned by DeepCC captured biological characteristics associated with distinct molecular subtypes"
- [methods] By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra.: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra."
- [methods] # obtain deep features 
df <- get_DeepCC_features(deepcc_model, fs): "# obtain deep features 
df <- get_DeepCC_features(deepcc_model, fs)"
- [readme] #use the deep feature for visualization of classification.: "#use the deep feature for visualization of classification. 
#Assume deep feature stored in df, labels gotten form `get_DeepCC_label` stored in labels,"
