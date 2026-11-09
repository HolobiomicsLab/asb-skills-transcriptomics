---
name: spatial-transcriptomics-network-inference
description: Use when building interpretable GRNs from whole-brain Visium or other
  spatial transcriptomics count matrices (e.g. mouse AD plaque/age data) and scoring
  network modules per spot for spatial or disease association.
when_to_use_negative:
- When the analysis requires the spatial coordinates themselves to inform network
  construction — SCING builds GRNs from expression only and 'the spatial information
  of spatial transcriptomics is not explicitly utilized in SCING GRN construction'
- When the input is bulk tissue data or very large single-cell datasets beyond the
  memory/practicality of building many subsampled GRNs — SCING relies on supercells
  and subsample bagging
- When causal or directional regulatory links are required — inferred links 'are not
  necessarily causal or directional'
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCING
  role: 'Core GRN inference: builds supercells, constructs 100 subsampled gradient-boosting
    GRNs, merges to a consensus (edges present in 20% of networks), and prunes edges/cycles
    with conditional mutual information'
  repo: https://github.com/XiaYangLabOrg/SCING
- name: SCENIC
  role: Provides the AUCell method used to compute module-specific expression scores
    per spatial spot from the Leiden-partitioned GRN subnetworks
- name: enrichr
  role: Pathway enrichment analysis on the genes in each Leiden subnetwork to biologically
    annotate GRN modules
- name: ppcor
  role: Benchmarked baseline GRN inference method compared against SCING
- name: GRNBOOST2
  role: Benchmarked gradient-boosting baseline whose approach SCING extends with bagging-based
    consensus and edge pruning
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:b4356c3f2ce79c56200401837e3885648c9035f4721069a0dc51e688258fdc65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - GRNBOOST2
  - SCENIC
  - SCING
  - enrichr
  - ppcor
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spatial-transcriptomics-network-inference

## Summary

Infers gene regulatory networks (GRNs) from whole-slide Visium spatial transcriptomics data using SCING's bagging-based consensus of gradient-boosted networks, then partitions the merged GRN into Leiden subnetwork modules scored per spatial spot with AUCell for regional and phenotype association.

## When to use

Use when building interpretable GRNs from whole-brain Visium or other spatial transcriptomics count matrices (e.g. mouse AD plaque/age data) and scoring network modules per spot for spatial or disease association.

## When NOT to use

- When the analysis requires the spatial coordinates themselves to inform network construction — SCING builds GRNs from expression only and 'the spatial information of spatial transcriptomics is not explicitly utilized in SCING GRN construction'
- When the input is bulk tissue data or very large single-cell datasets beyond the memory/practicality of building many subsampled GRNs — SCING relies on supercells and subsample bagging
- When causal or directional regulatory links are required — inferred links 'are not necessarily causal or directional'

## Inputs

- Spatial transcriptomics count matrix (e.g. 10x Visium whole-brain data, GSE152506)
- Spot-level metadata including spatial coordinates, brain subregion annotations, phenotype staining (e.g. beta-amyloid plaque), and age/covariates
- Optional phenotype variable to include as a node in the GRN (e.g. plaque staining)

## Outputs

- Merged and edge-pruned consensus GRN (network of gene-gene edges plus phenotype nodes)
- Leiden-partitioned GRN subnetworks/modules (e.g. 33 subnetworks)
- Per-spot AUCell module-specific expression scores
- Module pathway enrichments (enrichr) and Pearson correlation statistics of module scores vs. plaque burden/age against a 1,000 random-gene-set null with Bonferroni correction
- Spatial maps of module expression and correlation significance (e.g. Figure 6-style visualizations)

## How to apply

Preprocess the spatial count matrix (e.g. GSE152506 Visium spots with beta-amyloid plaque staining annotations and mouse ages 3-18 months) and, if relevant, include a phenotype such as plaque staining as a node. Build supercells from Leiden-merged subclusters, then construct 100 GRNs in parallel on subsamples of supercells, identifying edges via gradient boosting regressors and keeping the consensus as edges appearing in at least 20% of the networks; merge and prune edges and cycles using conditional mutual information. Partition the merged GRN into subnetworks with the Leiden algorithm (e.g. 33 modules in the mouse AD application), score module-specific expression for each spatial spot with the AUCell method from SCENIC, and annotate modules with enrichr pathway enrichment. Assess regional specificity with t-tests/ANOVA across annotated brain subregions and quantify plaque/age associations by Pearson correlation against a null of 1,000 random gene sets with Bonferroni correction.

## Related tools

- **SCING** (Core GRN inference: builds supercells, constructs 100 subsampled gradient-boosting GRNs, merges to a consensus (edges present in 20% of networks), and prunes edges/cycles with conditional mutual information) — https://github.com/XiaYangLabOrg/SCING
- **SCENIC** (Provides the AUCell method used to compute module-specific expression scores per spatial spot from the Leiden-partitioned GRN subnetworks)
- **enrichr** (Pathway enrichment analysis on the genes in each Leiden subnetwork to biologically annotate GRN modules)
- **ppcor** (Benchmarked baseline GRN inference method compared against SCING)
- **GRNBOOST2** (Benchmarked gradient-boosting baseline whose approach SCING extends with bagging-based consensus and edge pruning)

## Examples

```
git clone https://github.com/XiaYangLabOrg/SCING.git; conda env create -n scing --file install/scing.environment.yml; conda activate scing; pip install pyitlib; pip install -e .; pip install pyscenic
```

## Evaluation signals

- Consensus edge threshold respected: final GRN contains only edges recovered in at least 20% of the 100 subsampled networks, with edges and cycles pruned via conditional mutual information
- Leiden partitioning yields well-defined subnetworks (e.g. 33 modules in the mouse AD Visium application) covering the merged network's genes
- Module-level correlation significance reproduced against the null: Pearson correlation of module AUCell scores vs. plaque burden compared to 1,000 random gene sets with Bonferroni-corrected significance (e.g. six modules significant, module 25 microglial/lysosome/migration-enriched and highly plaque-associated)
- Regional specificity validated: t-tests/ANOVA show module expression differs significantly across annotated brain subregions
- GRN quality checks: goodness-of-fit via cosine similarity of held-out gene expression predicted by gene-parent GBRs, and disease subnetwork modeling performance versus random GRNs via the Huang et al. random walk framework

## Limitations

- Inferred GRN links are not necessarily causal or directional
- Many hyperparameters require selection, and selection can be dataset dependent even though SCING shows robustness across a range of values
- Spatial information is not used during network construction; explicitly leveraging it is deferred to future work
- Additional omics modalities (scATAC-seq, scHi-C, ChIP-seq, cis/trans eQTL) are not yet incorporated and could improve GRN accuracy

## Evidence

- [readme] BuildNetwork.ipynb: Build Supercells, build many GRNs based on subsamples of the data, merge many GRNs to a final GRN plus prune edges.: "BuildNetwork.ipynb: Build Supercells, build many GRNs based on subsamples of the data, merge many GRNs to a final GRN plus prune edges."
- [readme] conda env create command from README setup: "conda env create -n scing --file install/scing.environment.yml"
- [methods] Application to whole brain visium data: "We apply SCING to human prefrontal cortex snRNAseq data with AD and Control patients, whole brain visium data, for AD vs WT mice at different ages"
- [discussion] Spatial information not used in construction: "the spatial information of spatial transcriptomics is not explicitly utilized in SCING GRN construction despite the fact that the resulting GRNs show spatial patterns"
- [readme] Consensus edge threshold of 20% of networks: "We find the consensus as edges that show up in 20% of the networks. We then prune edges and cycles using conditional mutual information"
