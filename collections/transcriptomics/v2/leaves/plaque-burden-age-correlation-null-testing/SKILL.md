---
name: plaque-burden-age-correlation-null-testing
description: Use when GRN modules from spatial transcriptomics (e.g., GSE152506 Visium
  AD vs WT mouse brain, 3-18 months) need to be tested for correlation with a continuous
  phenotype such as beta-amyloid plaque staining or age against a permutation null.
when_to_use_negative:
- When the GRN should explicitly incorporate spatial coordinates during construction
  — SCING does not use spatial information when building the network
- When the input is raw counts without a merged, pruned GRN and leiden-partitioned
  module gene sets — module scoring cannot be performed
- When testing categorical group differences rather than correlations with continuous
  phenotypes like plaque burden or age
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SCING
  role: Builds supercells, infers 100 bagged GRNs from subsamples, merges and prunes
    them into the final GRN whose leiden modules are correlated with plaque burden
    and age
  repo: https://github.com/XiaYangLabOrg/SCING
- name: SCENIC
  role: Provides the AUCell method used to compute module-specific expression scores
    for each spot/cell prior to phenotype correlation
  repo: https://github.com/XiaYangLabOrg/SCING
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:bedfc0b2299fad9446b109c44acfcca758d3e1a1599eb3c1e72f3b628e2aceeb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - SCENIC
  - SCING
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# plaque-burden-age-correlation-null-testing

## Summary

Test SCING GRN modules inferred from mouse Alzheimer's disease Visium spatial transcriptomics for significant association with amyloid beta plaque burden and mouse age, using Pearson correlation of AUCell module scores against a 1,000 random gene-set permutation null with Bonferroni correction.

## When to use

Use when GRN modules from spatial transcriptomics (e.g., GSE152506 Visium AD vs WT mouse brain, 3-18 months) need to be tested for correlation with a continuous phenotype such as beta-amyloid plaque staining or age against a permutation null.

## When NOT to use

- When the GRN should explicitly incorporate spatial coordinates during construction — SCING does not use spatial information when building the network
- When the input is raw counts without a merged, pruned GRN and leiden-partitioned module gene sets — module scoring cannot be performed
- When testing categorical group differences rather than correlations with continuous phenotypes like plaque burden or age

## Inputs

- GSE152506 Visium spatial transcriptomics count matrix (whole brain, AD vs WT mice, ages 3-18 months)
- Spot-level metadata including beta-amyloid plaque staining annotations and mouse age
- Merged, edge-pruned SCING GRN (with plaque staining included as a node)
- Leiden-partitioned GRN subnetworks / module gene sets

## Outputs

- Per-module AUCell expression scores for each spatial spot
- Pearson correlation coefficients of each module with plaque burden and age
- Permutation null distributions (1,000 random gene sets per module)
- Bonferroni-corrected significance calls (e.g., six significant plaque-associated modules; module 25 flagged as highly plaque-associated)
- Figure 6-style spatial maps of module expression with correlation significance annotations

## How to apply

First build supercells from the Visium count matrix, construct 100 GRNs from subsamples in parallel with SCING's BuildNetwork workflow, merge them into a final GRN, and prune edges including plaque staining as a network node. Partition the merged GRN (e.g., into 33 subnetworks) with the leiden graph partitioning algorithm, then score module-specific expression per spatial spot with the AUCell method from SCENIC. Optionally first test regional specificity of module expression across annotated brain subregions with t-tests/ANOVA. For each module, compute Pearson correlation of its spot-level expression with plaque burden and with age, comparing against a null of 1,000 random gene sets matched from the same data, and apply Bonferroni correction for multiple testing across modules. Correct application yields modules like module 25 (microglial activation, lysosome, cell migration) emerging as highly plaque-associated, consistent with expected AD pathogenesis.

## Related tools

- **SCING** (Builds supercells, infers 100 bagged GRNs from subsamples, merges and prunes them into the final GRN whose leiden modules are correlated with plaque burden and age) — https://github.com/XiaYangLabOrg/SCING
- **SCENIC** (Provides the AUCell method used to compute module-specific expression scores for each spot/cell prior to phenotype correlation) — https://github.com/XiaYangLabOrg/SCING

## Examples

```
conda env create -n scing --file install/scing.environment.yml && conda activate scing && pip install pyscenic  # then run BuildNetwork.ipynb followed by ModuleBasedDimensionalityReduction.ipynb
```

## Evaluation signals

- The merged GRN contains plaque staining as a node and was partitioned into 33 leiden subnetworks
- Every module has a Pearson correlation value plus a null distribution from 1,000 random gene sets
- p-values are Bonferroni-corrected across all modules before calling significance
- Approximately six modules show significant plaque-burden association, and top modules annotate to expected AD biology (microglial activation, lysosome, cell migration via enrichr)
- Module spatial maps reproduce expected regional patterns consistent with the spot-level plaque staining annotations

## Limitations

- SCING does not utilize spatial information during network construction, even though the resulting GRNs show spatial patterns; leveraging it explicitly is deferred to future work
- Inferred GRN links are not necessarily causal or directional, so module-phenotype correlations reflect association, not causation
- Many hyperparameters require selection and selection can be dataset dependent, affecting the modules available for correlation testing

## Evidence

- [methods] leiden partition + AUCell scoring: "We apply the leiden graph partitioning algorithm to identify GRN subnetworks"
- [methods] AUCell module scores per cell: "We combine these subnetworks with the AUCell method to get module specific expression for each cell"
- [readme] README workflow notebook for module scores: "ModuleBasedDimensionalityReduction.ipynb: Take the merged network, and divide it into subnetworks with the leiden graph partitioning algorithm, finally run the AUCell approach to get module scores"
- [other] task card finding: six plaque-associated modules: [evidence span withheld — anchor status: not_found]
- [discussion] SCING spatial limitation: "the spatial information of spatial transcriptomics is not explicitly utilized in SCING GRN construction despite the fact that the resulting GRNs show spatial patterns"
