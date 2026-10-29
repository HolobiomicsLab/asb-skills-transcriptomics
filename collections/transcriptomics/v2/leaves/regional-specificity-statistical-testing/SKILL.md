---
name: regional-specificity-statistical-testing
description: Use when spatial transcriptomics spot-level module scores (e.g., AUCell
  scores per Visium spot) need testing for regional specificity across annotated anatomical
  subregions or correlation with spatially mapped phenotypes such as plaque burden
  or age.
when_to_use_negative:
- The GRN has not yet been built and leiden-partitioned into modules (SCING BuildNetwork
  and leiden partitioning must precede AUCell scoring and testing)
- Spot-level spatial annotations or phenotype measures (e.g., plaque staining) are
  unavailable, since regional specificity and correlation tests require them
- Single-cell (non-spatial) data with no anatomical region or spatial phenotype annotations,
  where regional specificity testing is not meaningful
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SCING
  role: Provides the merged, edge-pruned GRN and leiden-partitioned subnetworks whose
    module scores are tested; plaque staining is included as a node in the network
  repo: github.com/XiaYangLabOrg/SCING
- name: SCENIC
  role: Supplies the AUCell method used to compute module-specific expression scores
    for each spatial spot
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:755f2e3629910691e854046105afc55c440831e8ef2b06341339b48430bd2e0c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCENIC
  - SCING
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Regional-specificity statistical testing

## Summary

Statistically tests whether SCING GRN module expression scores differ across annotated brain subregions and whether they correlate with phenotypes (amyloid beta plaque burden, age) in mouse AD Visium spatial transcriptomics, using t-tests/ANOVA plus Pearson correlations against permutation nulls with Bonferroni correction.

## When to use

Use when spatial transcriptomics spot-level module scores (e.g., AUCell scores per Visium spot) need testing for regional specificity across annotated anatomical subregions or correlation with spatially mapped phenotypes such as plaque burden or age.

## When NOT to use

- The GRN has not yet been built and leiden-partitioned into modules (SCING BuildNetwork and leiden partitioning must precede AUCell scoring and testing)
- Spot-level spatial annotations or phenotype measures (e.g., plaque staining) are unavailable, since regional specificity and correlation tests require them
- Single-cell (non-spatial) data with no anatomical region or spatial phenotype annotations, where regional specificity testing is not meaningful

## Inputs

- GSE152506 Visium count matrix and spot metadata with beta-amyloid plaque staining annotations and mouse ages (3-18 months)
- Merged, edge-pruned SCING GRN partitioned into 33 leiden subnetworks
- Annotated brain subregion labels for each spatial spot

## Outputs

- Per-spot AUCell module-specific expression scores for each GRN module
- Regional specificity test statistics (t-tests/ANOVA) across brain subregions per module
- Pearson correlation coefficients of module expression with plaque burden and age, with Bonferroni-corrected significance vs. 1,000 random-gene-set nulls
- Figure 6-style spatial maps of module expression and correlation significance

## How to apply

First compute module-specific expression per spatial spot with AUCell (SCENIC) on the leiden-partitioned SCING GRN subnetworks, with plaque staining included as a node in the merged GRN. Test regional specificity of module expression across annotated brain subregions with t-tests and ANOVA. For phenotype association, compute Pearson correlations of each module's expression with plaque burden and mouse age (3–18 months), evaluating significance against a null of 1,000 random gene sets and applying Bonferroni correction for multiple testing across the 33 modules. Interpret significant plaque-associated modules (e.g., module 25, enriched for microglial activation, lysosome, and cell migration) via pathway enrichment of module genes with enrichr.

## Related tools

- **SCING** (Provides the merged, edge-pruned GRN and leiden-partitioned subnetworks whose module scores are tested; plaque staining is included as a node in the network) — github.com/XiaYangLabOrg/SCING
- **SCENIC** (Supplies the AUCell method used to compute module-specific expression scores for each spatial spot)

## Examples

```
conda activate scing && pip install pyscenic; jupyter notebook tutorials/ModuleBasedDimensionalityReduction.ipynb
```

## Evaluation signals

- Exactly 33 GRN subnetworks result from leiden partitioning of the merged GRN, each with AUCell scores computed for every spot
- Plaque burden and age correlations are evaluated against a null distribution of 1,000 random gene sets, and only modules surviving Bonferroni correction are called significant
- Six modules should be significantly associated with plaque burden, and module 25 (microglial activation, lysosome, cell migration enrichment) should be highly plaque-associated
- Regional specificity should reproduce expected biology: plaque-associated module scores differ significantly across annotated brain subregions (t-test/ANOVA p < 0.05)
- Spatial maps (Figure 6-style) show plaque-associated module expression co-localizing with amyloid beta plaque staining annotations

## Limitations

- Inferred GRN links are not necessarily causal or directional, so module-phenotype associations are correlational
- Many hyperparameters require selection and selection can be dataset dependent
- Spatial information of spatial transcriptomics is not explicitly utilized in GRN construction; the resulting GRNs show spatial patterns but SCING does not model space during inference
- Explicitly leveraging spatial information in network construction is deferred to future work
