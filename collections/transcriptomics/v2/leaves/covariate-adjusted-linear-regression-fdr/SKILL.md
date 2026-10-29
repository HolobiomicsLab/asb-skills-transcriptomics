---
name: covariate-adjusted-linear-regression-fdr
description: Use when you have per-cell module-specific expression scores from a GRN
  (e.g. SCING Leiden modules scored with AUCell) plus per-cell clinical metadata (trait
  labels and confounders such as sex), and need FDR-controlled module-trait association
  statistics.
when_to_use_negative:
- You do not yet have per-cell module scores — the module-level expression must first
  be computed (e.g. Leiden partitioning of the merged SCING GRN plus AUCell), so applying
  the regression to raw per-gene counts is out of scope
- Your question is about causal or directional regulatory effects — the article notes
  SCING GRN links 'are not necessarily causal or directional', so regression associations
  are correlative, not causal
- Your traits have no possible confounders to adjust and only a single response variable
  — a covariate-adjusted multi-trait regression framework is unnecessary overhead
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SCING
  role: Provides the merged, edge-pruned GRN whose Leiden subnetworks define the gene
    modules being tested
  repo: https://github.com/XiaYangLabOrg/SCING
- name: SCENIC
  role: Its AUCell method (pyscenic) computes the module-specific expression scores
    per cell that serve as the regression response variable
  repo: https://github.com/XiaYangLabOrg/SCING
- name: leiden
  role: Graph partitioning algorithm applied to the merged SCING GRN to produce the
    subnetwork modules
  repo: https://github.com/XiaYangLabOrg/SCING
- name: enrichR
  role: Downstream annotation of significant modules via pathway enrichment against
    GO biological process, DisGeNET, Reactome, BioCarta, and KEGG
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:a799a70a572a0b36d04f8bc2393316a80873b59993a8e51a47fc2f8d4477e70b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - SCENIC
  - SCING
  - enrichR
  - leiden
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Covariate-adjusted linear regression with Benjamini-Hochberg FDR for module-trait association

## Summary

Tests whether SCING GRN module scores (computed per cell via Leiden subnetworks and AUCell) are significantly associated with multiple disease traits — here Alzheimer's disease diagnosis, plaque stage, and tangle stage — by linear regression with sex regressed out as a covariate and Benjamini-Hochberg FDR correction across modules.

## When to use

Use when you have per-cell module-specific expression scores from a GRN (e.g. SCING Leiden modules scored with AUCell) plus per-cell clinical metadata (trait labels and confounders such as sex), and need FDR-controlled module-trait association statistics.

## When NOT to use

- You do not yet have per-cell module scores — the module-level expression must first be computed (e.g. Leiden partitioning of the merged SCING GRN plus AUCell), so applying the regression to raw per-gene counts is out of scope
- Your question is about causal or directional regulatory effects — the article notes SCING GRN links 'are not necessarily causal or directional', so regression associations are correlative, not causal
- Your traits have no possible confounders to adjust and only a single response variable — a covariate-adjusted multi-trait regression framework is unnecessary overhead

## Inputs

- Per-cell module-specific expression scores (AUCell module scores from Leiden subnetworks of the merged SCING GRN)
- Per-cell clinical metadata table with diagnosis, plaque stage, tangle stage, and sex (e.g. from snRNAseq prefrontal cortex AD/Control data)

## Outputs

- Linear regression statistics (effect estimates and p-values) per module per trait, with Benjamini-Hochberg FDR-corrected values across modules
- Set of disease-associated modules passing FDR < 0.05 for diagnosis, plaque stage, and/or tangle stage
- module_pathway_annotations.csv combining module assignments, AUCell scores, regression statistics, and enrichR pathway annotations

## How to apply

First compute module scores for every cell by combining Leiden-partitioned GRN subnetworks with the AUCell method from SCENIC (pyscenic), as described in ModuleBasedDimensionalityReduction. Then, for each module, run a linear regression of the module score on each trait of interest (diagnosis, plaque stage, tangle stage) while regressing out the covariate sex. Apply Benjamini-Hochberg correction across modules at FDR < 0.05 as the significance threshold for calling a module disease-associated. In the SCING microglia application, ~43% of modules were significantly (FDR < 0.05) associated with at least one AD trait and ~24% with all three, and significant modules were passed forward to enrichR pathway enrichment (GO biological process, DisGeNET, Reactome, BioCarta, KEGG) per PathwayEnrichmentOfModule. Assemble module assignments, AUCell scores, and regression statistics into module_pathway_annotations.csv.

## Related tools

- **SCING** (Provides the merged, edge-pruned GRN whose Leiden subnetworks define the gene modules being tested) — https://github.com/XiaYangLabOrg/SCING
- **SCENIC** (Its AUCell method (pyscenic) computes the module-specific expression scores per cell that serve as the regression response variable) — https://github.com/XiaYangLabOrg/SCING
- **leiden** (Graph partitioning algorithm applied to the merged SCING GRN to produce the subnetwork modules) — https://github.com/XiaYangLabOrg/SCING
- **enrichR** (Downstream annotation of significant modules via pathway enrichment against GO biological process, DisGeNET, Reactome, BioCarta, and KEGG)

## Evaluation signals

- Benjamini-Hochberg corrected p-values across modules satisfy FDR < 0.05 for the reported significant modules
- Every module's regression includes sex as a regressed-out covariate; models without covariate adjustment are incomplete
- Regression is run per trait (diagnosis, plaque stage, tangle stage) so each module yields statistics for all three AD traits
- For the SCING microglia reference application, roughly 43% of modules are significant for at least one trait and ~24% for all three — large deviations warrant checking module score computation and metadata alignment
- Output table (module_pathway_annotations.csv) contains module assignments, AUCell scores, regression statistics, and pathway annotations for significant modules

## Limitations

- SCING GRN links are not necessarily causal or directional, so module-trait regression findings are associative
- The regression significance threshold is dataset-dependent and multiple-testing burden scales with the number of Leiden modules detected
- Many SCING hyperparameters (e.g. edge consensus and pruning settings upstream of module detection) require selection and can be dataset dependent

## Evidence

- [readme] We combine these subnetworks with the AUCell method to get module specific expression for each cell and further combine the gene modules with pathway knowledge bases: "We combine these subnetworks with the AUCell method to get module specific expression for each cell and further combine the gene modules with pathway knowledge bases"
- [readme] These can be used for clustering or phenotypic association.: "These can be used for clustering or phenotypic association."
- [intro] These modules can be used for clustering, phenotypic association, and biological annotation through: "These modules can be used for clustering, phenotypic association, and biological annotation through"
- [discussion] the links between genes are not necessarily causal or directional: "the links between genes are not necessarily causal or directional"
