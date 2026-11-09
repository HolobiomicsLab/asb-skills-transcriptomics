---
name: metabolic-model-essentiality-analysis
description: Use when you have context-specific metabolic models of infected vs. mock
  tissue (built via rFASTCORMICS from GEO datasets such as GSE147507/GSE148729) and
  need to enumerate essential genes or synergistic double-knockout gene pairs to prioritize
  drug targets.
when_to_use_negative:
- When the goal is directly docking drugs on the viral proteome — DCcov notes identified
  targets come from 'network effects on the host metabolome (as the virus is only
  modeled through its biomass function) rather than direct docking'
- When input models lack the viral biomass objective function or are mock (uninfected)
  models, since essentiality is scored against viral biomass production
- When only differential expression is needed — use DESeq2 (|log2FC| > 1, adjusted
  p < 0.05) instead of knockout simulation
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: COBRA Toolbox V3
  role: Provides the doubleGeneDeletion routine used to systematically knock out all
    gene pairs and score viral biomass flux
  repo: https://github.com/opencobra/cobratoolbox
- name: MATLAB
  role: Execution environment for the COBRA Toolbox knockout analyses
- name: rFASTCORMICS
  role: Reconstructs the infected context-specific models on which the gene deletion
    analysis is performed
  repo: https://github.com/sysbiolux/rFASTCORMICS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:1ca743f0958b1657df85fcc781e6b821e6e92c29ce2de204beb10e27f9ce2b92
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sysbiolux/DCcov
  repo_url: https://github.com/opencobra/cobratoolbox
  tools:
  - COBRA Toolbox
  - COBRA Toolbox V3
  - MATLAB
  - rFASTCORMICS
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# metabolic-model-essentiality-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Systematic in silico single- and double-gene deletion on context-specific genome-scale metabolic models carrying a viral biomass objective function (VBOF) to identify essential genes and synergistic gene pairs whose knockout reduces viral biomass production without affecting host biomass maintenance.

## When to use

Use when you have context-specific metabolic models of infected vs. mock tissue (built via rFASTCORMICS from GEO datasets such as GSE147507/GSE148729) and need to enumerate essential genes or synergistic double-knockout gene pairs to prioritize drug targets.

## When NOT to use

- When the goal is directly docking drugs on the viral proteome — DCcov notes identified targets come from 'network effects on the host metabolome (as the virus is only modeled through its biomass function) rather than direct docking'
- When input models lack the viral biomass objective function or are mock (uninfected) models, since essentiality is scored against viral biomass production
- When only differential expression is needed — use DESeq2 (|log2FC| > 1, adjusted p < 0.05) instead of knockout simulation

## Inputs

- Context-specific infected lung metabolic models with VBOF (COBRA format, reconstructed with rFASTCORMICS from GSE147507 and GSE148729 expression data)
- Consistent genome-scale reconstructions (Recon2.04 or RECON3D_01) as model base
- Gene-to-reaction mapping (GPR rules / dico table for identifier mapping)
- Viral biomass objective function definition

## Outputs

- essential_gene_pairs.csv containing 598 unique essential gene pairs
- Classified pair lists: non-essential synergistic pairs and essential-plus-non-essential pairs per infected model
- Per-pair viral biomass production flux scores
- Druggable subset (56 druggable gene pairs in DCcov, mapped to DrugBank V5)

## How to apply

Load the infected context-specific genome-scale models (carrying the VBOF from Renz et al., 2020) into MATLAB with the COBRA Toolbox V3, using Recon2.04 or RECON3D_01 as the reconstruction base. Run the doubleGeneDeletion routine on each infected model to systematically knock out all gene pairs and score each pair by the resulting viral biomass production flux. Classify knockout results into non-essential synergistic pairs (each single knockout viable, double knockout abolishes viral biomass) and essential-plus-non-essential pairs (one essential gene paired with a non-essential gene). Concatenate the two pair classes across all infected models from the severity (GSE147507) and time-series (GSE148729) studies and deduplicate to obtain the unique essential gene pair set. Validate by confirming the exact count of unique pairs (598 in the DCcov study) and that each listed pair reduces viral biomass production without affecting host biomass maintenance; downstream, map pairs to DrugBank V5 for drug combination repositioning.

## Related tools

- **COBRA Toolbox V3** (Provides the doubleGeneDeletion routine used to systematically knock out all gene pairs and score viral biomass flux) — https://github.com/opencobra/cobratoolbox
- **MATLAB** (Execution environment for the COBRA Toolbox knockout analyses)
- **rFASTCORMICS** (Reconstructs the infected context-specific models on which the gene deletion analysis is performed) — https://github.com/sysbiolux/rFASTCORMICS

## Examples

```
[grRatioD, grRatioSD, geneRxnsD, geneRxnsSD] = doubleGeneDeletion(infected_model_with_VBOF);
```

## Evaluation signals

- The concatenated, deduplicated gene pair set contains exactly 598 unique essential gene pairs across the severity and time-series studies
- Each listed pair reduces viral biomass production flux while host biomass maintenance is unaffected
- Single knockouts in non-essential synergistic pairs remain viable; only the double knockout abolishes viral biomass
- Of the 598 pairs, 56 gene pairs are druggable (map to DrugBank V5 entries)
- 56 of the pairs are accounted for: gene pairs where both genes are targeted by the same drug are moved to the single-drug candidate list

## Limitations

- Further in vitro single- and double-gene deletion studies are needed to determine the essentiality of the identified single genes and gene pairs
- Analysis is limited by the modeled lung cell lines (A549, Calu-3, H1299, NHBE)
- Drug targets reflect network effects on the host metabolome, as the virus is only modeled through its biomass function, not direct docking on the viral proteome

## Evidence

- [methods] Source card workflow: DKO routine: [evidence span withheld — anchor status: not_found]
- [results] Source card finding: 598 pairs, 56 druggable: [evidence span withheld — anchor status: not_found]
- [readme] README: COBRA Toolbox scope: [evidence span withheld — anchor status: external_doc]
- [readme] README: rFASTCORMICS input models: [evidence span withheld — anchor status: external_doc]
