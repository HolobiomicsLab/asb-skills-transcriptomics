---
name: pathway-enrichment-annotation-of-gene-modules
description: Use when you have gene subnetworks/modules partitioned from a gene regulatory
  network (e.g., SCING GRN modules via Leiden) and need biological annotation or disease-relevant
  pathway interpretation for each module.
when_to_use_negative:
- Input is an unpartitioned GRN or raw expression matrix — the Leiden module partitioning
  (ModuleBasedDimensionalityReduction) must be run first to obtain per-module gene
  sets
- You need de novo motif/TFBS enrichment — SCING's companion steps prune edges via
  TFBS and conditional mutual information; this skill only queries curated pathway
  knowledge bases via enrichR
- Spatial-resolved pathway mapping is required — SCING does not explicitly utilize
  spatial information during network construction, so enrichment is on module gene
  sets, not spatial coordinates
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: enrichR
  role: R package used to run pathway enrichment analysis on the genes in each Leiden-defined
    subnetwork against GO Biological Process, DisGeNET, Reactome, BioCarta, and KEGG
    knowledge bases
  repo: github.com/XiaYangLabOrg/SCING
- name: SCING
  role: Produces the merged, edge-pruned GRN whose Leiden-partitioned subnetworks
    supply the module gene lists for enrichment
  repo: github.com/XiaYangLabOrg/SCING
- name: SCENIC
  role: Provides the AUCell method (via pyscenic) used alongside module partitioning
    to compute module-specific expression scores that accompany the enrichment annotations
  repo: github.com/XiaYangLabOrg/SCING
- name: conda
  role: Used to create the dedicated enrichR environment (install/pathway.environment.yml)
    required for the enrichment step
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:df6fde43831553c64ba376ef4c8867d18166a23c5a2840953d0250ae468cea20
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCENIC
  - SCING
  - conda
  - enrichR
  - leiden
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pathway-enrichment-annotation-of-gene-modules

## Summary

Annotates gene modules (Leiden subnetworks from a SCING-inferred GRN) with biological pathways by running enrichment analysis of each module's gene set against knowledge bases such as GO Biological Process, DisGeNET, Reactome, BioCarta, and KEGG using enrichR in R.

## When to use

Use when you have gene subnetworks/modules partitioned from a gene regulatory network (e.g., SCING GRN modules via Leiden) and need biological annotation or disease-relevant pathway interpretation for each module.

## When NOT to use

- Input is an unpartitioned GRN or raw expression matrix — the Leiden module partitioning (ModuleBasedDimensionalityReduction) must be run first to obtain per-module gene sets
- You need de novo motif/TFBS enrichment — SCING's companion steps prune edges via TFBS and conditional mutual information; this skill only queries curated pathway knowledge bases via enrichR
- Spatial-resolved pathway mapping is required — SCING does not explicitly utilize spatial information during network construction, so enrichment is on module gene sets, not spatial coordinates

## Inputs

- Gene lists for each Leiden subnetwork module derived from the merged, edge-pruned SCING GRN
- Merged, edge-pruned GRN produced by the BuildNetwork step (BuildNetwork.ipynb output)
- Optionally, AUCell module scores and regression statistics (diagnosis, plaque stage, tangle stage, sex covariate) to assemble into module_pathway_annotations.csv

## Outputs

- Pathway enrichment terms per module from enrichR against GO Biological Process, DisGeNET, Reactome, BioCarta, and KEGG
- module_pathway_annotations.csv combining module assignments, AUCell scores, regression statistics, and pathway annotations

## How to apply

First partition the merged, edge-pruned GRN into subnetwork modules using the Leiden graph partitioning algorithm (ModuleBasedDimensionalityReduction step), then take the gene list of each subnetwork as input to pathway enrichment. Install the dedicated R environment (install/pathway.environment.yml) and use the enrichR package to query each module's genes against GO Biological Process, DisGeNET, Reactome, BioCarta, and KEGG knowledge bases, following the PathwayEnrichmentOfModule procedure. For the AD microglia application, enrichment results are combined with module assignments, AUCell scores, and regression statistics (linear models of module score on diagnosis, plaque stage, and tangle stage with sex as covariate, Benjamini-Hochberg FDR < 0.05) into module_pathway_annotations.csv. Modules whose genes return terms from the specified knowledge bases are accepted as biologically annotated.

## Related tools

- **enrichR** (R package used to run pathway enrichment analysis on the genes in each Leiden-defined subnetwork against GO Biological Process, DisGeNET, Reactome, BioCarta, and KEGG knowledge bases) — github.com/XiaYangLabOrg/SCING
- **SCING** (Produces the merged, edge-pruned GRN whose Leiden-partitioned subnetworks supply the module gene lists for enrichment) — github.com/XiaYangLabOrg/SCING
- **SCENIC** (Provides the AUCell method (via pyscenic) used alongside module partitioning to compute module-specific expression scores that accompany the enrichment annotations) — github.com/XiaYangLabOrg/SCING
- **conda** (Used to create the dedicated enrichR environment (install/pathway.environment.yml) required for the enrichment step)

## Examples

```
conda env create -n enrichr --file install/pathway.environment.yml; conda activate enrichr; # then run PathwayEnrichmentOfModule.ipynb to enrich each Leiden module's genes via enrichR
```

## Evaluation signals

- enrichR returns terms for each module from the specified knowledge bases (GO Biological Process, DisGeNET, Reactome, BioCarta, KEGG) — modules with returned terms are accepted as annotated
- Every module gene list originates from a Leiden subnetwork of the merged, edge-pruned SCING GRN (traceable back to the BuildNetwork output)
- module_pathway_annotations.csv contains, for each module: module assignment, AUCell scores, regression statistics (BH FDR < 0.05 for significant trait associations), and pathway annotations
- The dedicated conda environment (enrichr env from pathway.environment.yml) is active when the enrichment notebook is executed
- Disease-associated modules should retrieve disease-relevant pathways, consistent with the finding that SCING GRNs 'retrieve disease relevant genes and pathways'

## Limitations

- Inferred GRN links are not necessarily causal or directional, so pathway annotations reflect co-regulatory module membership rather than causal mechanisms
- Many SCING hyperparameters require selection and selection can be dataset dependent, which can change module gene sets and thus enrichment results
- Additional omics information (scATAC-seq, scHi-C, ChIP-seq, cis/trans eQTL) is not yet incorporated, which limits regulatory context available to the annotated modules

## Evidence

- [methods] Pathway enrichment analysis of modules with enrichr: "Run pathways enrichment analysis on the genes in each subnetowrk determined from the leiden algorithm"
- [readme] enrichr environment setup: "If you want to perform pathway enrichment analysis with enrichr you must install R and enrichr."
- [methods] Module partitioning and AUCell context: "We apply the leiden graph partitioning algorithm to identify GRN subnetworks"
- [intro] Purpose of module annotation: "SCING GRNs are then partitioned into modules, to compute module specific expression for each cell. These modules can be used for clustering, phenotypic association, and biological annotation through"
- [readme] PathwayEnrichmentOfModule notebook role: "PathwayEnrichmentOfModule.ipynb: Run pathways enrichment analysis on the genes in each subnetowrk determined from the leiden algorithm. This enables biological annotation of modules."
