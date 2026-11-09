---
name: functional-spectra-interpretation
description: 'Use when a trained DeepCC model''s deep features must be validated biologically:
  samples have CMS (or analogous) labels and you need to show deep-feature space is
  more compact than signature-gene space and that feature groups map to known subtype
  characteristics.'
when_to_use_negative:
- The goal is unsupervised discovery of new subtypes — DeepCC is built for classification,
  not clustering of cancer patients
- The model has not been trained and only raw expression is available — this skill
  interprets deep features of a trained ANN, not raw functional spectra
- Only a single new sample without a reference expression profile is available (single-sample
  prediction requires getFunctionalSpectrum with refExp, and has too few samples for
  silhouette analysis)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_3474
tools:
- name: DeepCC
  role: Provides load_DeepCC_model, get_DeepCC_features, getFunctionalSpectra, and
    vis_samples used to load the model, extract deep features, and visualize sample
    heatmaps
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: keras
  role: Loads the trained ANN from the .hdf5 file inside load_DeepCC_model
- name: ggplot2
  role: Rendering engine for vis_samples heatmaps of feature-group structure
- name: cowplot
  role: theme_set(theme_cowplot()) styling for vis_samples plots
- name: msigdbr
  role: Optional alternative MSigDB source via get_msigdbr() for generating functional
    spectra
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:5c32d686fbbee486db716495b13594d3518f912a6acbd555a76c53ab3fe4ab19
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
  - msigdbr
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# functional-spectra-interpretation

## Summary

Interpret the ten deep features extracted from the last hidden layer of a trained DeepCC ANN by assessing silhouette-based subtype compactness and clustering the absolute Pearson correlation matrix to map feature groups onto subtype biology (MSI, metabolic dysregulation, EMT).

## When to use

Use when a trained DeepCC model's deep features must be validated biologically: samples have CMS (or analogous) labels and you need to show deep-feature space is more compact than signature-gene space and that feature groups map to known subtype characteristics.

## When NOT to use

- The goal is unsupervised discovery of new subtypes — DeepCC is built for classification, not clustering of cancer patients
- The model has not been trained and only raw expression is available — this skill interprets deep features of a trained ANN, not raw functional spectra
- Only a single new sample without a reference expression profile is available (single-sample prediction requires getFunctionalSpectrum with refExp, and has too few samples for silhouette analysis)

## Inputs

- Trained/pre-loaded DeepCC model object (e.g., CRC_TCGA from load_DeepCC_model; .hdf5 + .RData files)
- Functional spectra matrix from getFunctionalSpectra (MSigDB v7, 22,596 gene sets) for TCGA-COADREAD samples
- Character vector of CMS labels for the samples (for silhouette grouping)
- 273-gene CMS signature expression matrix on the same samples (baseline comparison)

## Outputs

- Ten-column deep-feature matrix (last hidden layer) per sample
- Average silhouette widths per hidden-layer depth and per CMS label
- Wilcoxon signed-rank test result comparing deep-feature vs. 273-gene signature silhouettes (P < 0.01 expected)
- Absolute Pearson correlation matrix among deep features and its hierarchical clustering with three feature groups
- Heatmaps of the three feature groups (MSI, metabolic, EMT) rendered with vis_samples

## How to apply

Load the pre-trained CRC DeepCC model via load_DeepCC_model('CRC_TCGA') (keras::load_model_hdf5 on the .hdf5 plus .RData levels) and extract ten deep features with get_DeepCC_features(deepcc_model, fs) from functional spectra computed by getFunctionalSpectra (MSigDB v7, 22,596 gene sets, default). Quantify within-CMS compactness and between-subtype separation by computing average silhouette widths of samples grouped by CMS labels at each hidden-layer depth, then compare deep-feature silhouettes against 273-gene CMS signature silhouettes on the same samples with the Wilcoxon signed-rank test at P < 0.01. Compute the absolute Pearson correlation matrix among the ten deep features and hierarchically cluster it to obtain three feature groups. Annotate these groups against subtype-relevant biology (MSI, metabolic dysregulation, EMT) and render Fig. 3b-d style heatmaps with vis_samples(df, labels, color) using ggplot2/cowplot (theme_set(theme_cowplot())).

## Related tools

- **DeepCC** (Provides load_DeepCC_model, get_DeepCC_features, getFunctionalSpectra, and vis_samples used to load the model, extract deep features, and visualize sample heatmaps) — https://github.com/CityUHK-CompBio/DeepCC
- **keras** (Loads the trained ANN from the .hdf5 file inside load_DeepCC_model)
- **ggplot2** (Rendering engine for vis_samples heatmaps of feature-group structure)
- **cowplot** (theme_set(theme_cowplot()) styling for vis_samples plots)
- **msigdbr** (Optional alternative MSigDB source via get_msigdbr() for generating functional spectra)

## Examples

```
CRC_TCGA <- load_DeepCC_model("CRC_TCGA"); df <- get_DeepCC_features(CRC_TCGA$classifier, fs); theme_set(theme_cowplot()); vis_samples(df, labels, color)
```

## Evaluation signals

- Exactly three distinct feature groups emerge from hierarchical clustering of the absolute Pearson correlation matrix among the ten deep features
- The three groups map cleanly onto MSI, metabolic dysregulation, and EMT (Fig. 3b-d, Table S5)
- Average silhouette widths in deep-feature space are significantly higher than in the 273-gene CMS signature space (Wilcoxon signed-rank, P < 0.01)
- Deep-feature matrix has exactly ten columns, matching the last hidden layer of the loaded model
- vis_samples heatmaps show compact within-CMS and separated between-CMS sample distributions

## Limitations

- DeepCC's gene set prior knowledge must be comprehensive to avoid potential bias; characteristic signaling pathways for each subtype should be included in the gene set database
- DeepCC is designed for classification, not clustering — it cannot discover subtypes de novo
- The interpretation is anchored to a known subtyping system (e.g., CMS); it presupposes trusted sample labels for silhouette grouping

## Evidence

- [results] Clustering of absolute Pearson correlation coefficients between the ten deep features yielded three distinct feature groups that are highly correlated with MSI, metabolic dysregulation, and higher EMT: [evidence span withheld — anchor status: not_found]
- [abstract] deep features captured subtype biology: "deep features learned by DeepCC captured biological characteristics associated with distinct molecular subtypes, enabling more compact within-subtype distribution and between-subtype separation"
- [methods] obtain deep features: "# obtain deep features 
df <- get_DeepCC_features(deepcc_model, fs)"
- [readme] load CRC TCGA model: "# for CRC model trained with TCGA data
CRC_TCGA <- load_DeepCC_model("CRC_TCGA")"
- [methods] MSigDB v7 default: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra."
