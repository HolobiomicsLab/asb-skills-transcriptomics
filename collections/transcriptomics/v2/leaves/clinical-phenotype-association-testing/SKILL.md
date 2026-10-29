---
name: clinical-phenotype-association-testing
description: Use when a merged, edge-pruned SCING GRN and snRNAseq matrix with per-cell
  clinical metadata (diagnosis, plaque stage, tangle stage, sex) are available and
  the goal is to test which GRN modules are significantly associated with disease
  traits.
when_to_use_negative:
- No merged, edge-pruned SCING GRN exists yet — the leiden partitioning and AUCell
  scoring require the BuildNetwork output as input.
- Expression data lack per-cell clinical phenotype labels (diagnosis, plaque stage,
  tangle stage) or the sex covariate needed for regression adjustment.
- The goal is causal/directional regulatory inference rather than phenotypic association
  — inferred GRN links are not necessarily causal or directional.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3272
tools:
- name: SCING
  role: Provides the merged, edge-pruned gene regulatory network that is partitioned
    into Leiden modules for phenotypic association testing.
  repo: github.com/XiaYangLabOrg/SCING
- name: SCENIC (pyscenic AUCell)
  role: Computes module-specific expression (AUCell) scores for each cell from the
    Leiden subnetworks, which serve as the regression response variables.
- name: leiden
  role: Graph partitioning algorithm used to divide the merged GRN into subnetwork
    modules prior to AUCell scoring.
- name: enrichR
  role: R package used to run pathway enrichment analysis on genes of significant
    modules against GO BP, DisGeNET, Reactome, BioCarta, and KEGG.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:fb676d3debf115005f9d2e9f39773f3edc5d8a8591376d3043efbfe383b32b92
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCENIC
  - SCENIC (pyscenic AUCell)
  - SCING
  - enrichR
  - leiden
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# clinical-phenotype-association-testing

## Summary

Associates single-cell GRN module expression scores with clinical traits (e.g., AD diagnosis, plaque stage, tangle stage) via linear regression with covariate adjustment (sex) and Benjamini-Hochberg FDR correction, then annotates significant modules with pathway enrichment using enrichR against GO biological process, DisGeNET, Reactome, BioCarta, and KEGG.

## When to use

Use when a merged, edge-pruned SCING GRN and snRNAseq matrix with per-cell clinical metadata (diagnosis, plaque stage, tangle stage, sex) are available and the goal is to test which GRN modules are significantly associated with disease traits.

## When NOT to use

- No merged, edge-pruned SCING GRN exists yet — the leiden partitioning and AUCell scoring require the BuildNetwork output as input.
- Expression data lack per-cell clinical phenotype labels (diagnosis, plaque stage, tangle stage) or the sex covariate needed for regression adjustment.
- The goal is causal/directional regulatory inference rather than phenotypic association — inferred GRN links are not necessarily causal or directional.

## Inputs

- Merged, edge-pruned SCING GRN (output of BuildNetwork.ipynb)
- snRNAseq expression matrix (e.g., human prefrontal cortex snRNAseq)
- Per-cell clinical metadata: diagnosis, plaque stage, tangle stage, sex
- Pathway knowledge bases for enrichR: GO Biological Process, DisGeNET, Reactome, BioCarta, KEGG

## Outputs

- Leiden module (subnetwork) gene assignments
- AUCell module-specific expression scores for every cell
- Linear regression statistics per module per trait with Benjamini-Hochberg FDR-corrected p-values
- module_pathway_annotations.csv (module assignments, AUCell scores, regression statistics, pathway annotations)

## How to apply

Take the merged, edge-pruned microglia GRN produced by the BuildNetwork step and divide it into subnetworks with the leiden graph partitioning algorithm (ModuleBasedDimensionalityReduction procedure). Combine each Leiden subnetwork with the AUCell method from SCENIC (pyscenic) to compute module-specific expression scores for every cell in the dataset. Run linear regression of each module score on diagnosis, plaque stage, and tangle stage while regressing out sex as a covariate, and apply Benjamini-Hochberg correction across modules at FDR < 0.05. For significant modules, run pathway enrichment analysis on the subnetwork genes with enrichR in R (PathwayEnrichmentOfModule procedure) against GO biological process, DisGeNET, Reactome, BioCarta, and KEGG knowledge bases. In the SCING microglia AD application, roughly 43% of modules were significantly (FDR < 0.05) associated with at least one AD trait and roughly 24% with all three traits.

## Related tools

- **SCING** (Provides the merged, edge-pruned gene regulatory network that is partitioned into Leiden modules for phenotypic association testing.) — github.com/XiaYangLabOrg/SCING
- **SCENIC (pyscenic AUCell)** (Computes module-specific expression (AUCell) scores for each cell from the Leiden subnetworks, which serve as the regression response variables.)
- **leiden** (Graph partitioning algorithm used to divide the merged GRN into subnetwork modules prior to AUCell scoring.)
- **enrichR** (R package used to run pathway enrichment analysis on genes of significant modules against GO BP, DisGeNET, Reactome, BioCarta, and KEGG.)

## Examples

```
pip install pyscenic
```

## Evaluation signals

- A module is accepted as disease-associated only when its linear regression covariate p-values pass Benjamini-Hochberg FDR < 0.05 across modules.
- Module annotation is accepted only when enrichR returns terms from the specified pathway knowledge bases (GO biological process, DisGeNET, Reactome, BioCarta, KEGG).
- AUCell scores must be computed for every cell in the dataset for each Leiden subnetwork, not for a subsample of cells.
- For the human AD microglia application, expect roughly 43% of modules significantly associated with at least one trait (diagnosis, plaque stage, tangle stage) and roughly 24% with all three at FDR < 0.05.
- All three traits (diagnosis, plaque stage, tangle stage) must be tested with sex regressed out as a covariate; results should be reproducible from module_pathway_annotations.csv.

## Limitations

- SCING hyperparameters require selection and selection can be dataset dependent, which can affect the modules available for association testing.
- The links between genes in the GRN are not necessarily causal or directional, so module-trait associations are associative, not mechanistic.
- Running AUCell from SCENIC requires pyscenic, and pathway enrichment with enrichR requires a separate R environment (pathway.environment.yml).

## Evidence

- [intro] SCING GRNs are then partitioned into modules, to compute module specific expression for each cell. These modules can be used for clustering, phenotypic association, and biological annotation: "SCING GRNs are then partitioned into modules, to compute module specific expression for each cell. These modules can be used for clustering, phenotypic association, and biological annotation"
- [methods] We combine these subnetworks with the AUCell method to get module specific expression for each cell: "We combine these subnetworks with the AUCell method to get module specific expression for each cell"
- [readme] Run pathways enrichment analysis on the genes in each subnetowrk determined from the leiden algorithm. This enables biological annotation of modules.: "Run pathways enrichment analysis on the genes in each subnetowrk determined from the leiden algorithm. This enables biological annotation of modules."
- [readme] If you want to use the AUCell from SCENIC for graph based dimensionality reduction you must install pyscenic: "If you want to use the AUCell from SCENIC for graph based dimensionality reduction you must install pyscenic"
