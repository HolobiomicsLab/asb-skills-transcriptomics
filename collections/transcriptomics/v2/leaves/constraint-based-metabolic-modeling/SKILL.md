---
name: constraint-based-metabolic-modeling
description: Use when RNA-seq expression data from infected and mock (control) tissue/cell
  samples must be turned into context-specific metabolic models for knockout-based
  essentiality prediction and drug-target screening.
when_to_use_negative:
- The goal is direct docking or interaction modeling against the viral proteome —
  the virus is only represented through its biomass function, so drug targets reflect
  network effects on the host metabolome
- Tissue or cell types outside the modeled lung lines (A549, Calu-3, H1299, NHBE)
  without additional validation
- Only expression matrices without a consistent genome-scale reconstruction and gene-mapping
  dictionary are available
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: rFASTCORMICS
  role: Reconstructs context-specific metabolic models from RNA-seq expression data
    using the FASTCORMICS algorithm family
  repo: https://github.com/sysbiolux/rFASTCORMICS
- name: COBRA Toolbox V3
  role: MATLAB toolbox for loading models, running FBA, and executing singleGeneDeletion
    knockout analyses
  repo: https://github.com/opencobra/cobratoolbox
- name: Recon2.04
  role: Generic genome-scale reconstruction augmented with the VBOF as input for context-specific
    reconstruction
- name: Recon3D_01
  role: Generic genome-scale reconstruction augmented with the VBOF as input for context-specific
    reconstruction and pathway (subSystems) annotation
- name: MATLAB
  role: Execution environment for rFASTCORMICS and COBRA Toolbox workflows
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:890924c78e2e35a3cf20588849765cf6bb20a4e811373b9669e5edb8adc5ee64
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sysbiolux/DCcov
  repo_url: https://github.com/sysbiolux/rFASTCORMICS
  tools:
  - COBRA Toolbox
  - COBRA Toolbox V3
  - MATLAB
  - Recon2.04
  - Recon3D_01
  - rFASTCORMICS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# constraint-based-metabolic-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct context-specific genome-scale metabolic models from infected-lung expression data (Recon2.04/Recon3D_01 augmented with the viral biomass objective function) and use flux balance analysis in MATLAB/COBRA Toolbox to identify host genes whose knockout reduces viral biomass, enabling drug repositioning.

## When to use

Use when RNA-seq expression data from infected and mock (control) tissue/cell samples must be turned into context-specific metabolic models for knockout-based essentiality prediction and drug-target screening.

## When NOT to use

- The goal is direct docking or interaction modeling against the viral proteome — the virus is only represented through its biomass function, so drug targets reflect network effects on the host metabolome
- Tissue or cell types outside the modeled lung lines (A549, Calu-3, H1299, NHBE) without additional validation
- Only expression matrices without a consistent genome-scale reconstruction and gene-mapping dictionary are available

## Inputs

- RNA-seq expression data (FPKM or TPM transformed) for infected and mock samples, e.g. GSE147507 (severity study) and GSE148729 (time-series study)
- Genome-scale metabolic reconstructions Recon2.04 and Recon3D_01 (VMH format, COBRA format in MATLAB)
- Viral biomass objective function (VBOF) from BioModels MODEL2003020001
- Gene identifier dictionary (dico) mapping experiment rownames to model genes
- DEG calls from DESeq2 (absolute log2 fold change >1, adjusted p-value < 0.05)

## Outputs

- Context-specific infected and mock metabolic models (50 models: 28 infected, 22 mock; median 3646 metabolites, 2456.5 reactions)
- List of essential genes (e.g. essential_genes.csv with 23 unique genes incl. CRLS1, GUK1, SGMS1, DTYMK, CMPK1, TYMS)
- Essential gene pairs from double knockouts
- Repositioned single drugs and drug combinations (85 drugs, 52 combinations in DCcov)

## How to apply

Load Recon2.04 or Recon3D_01 in MATLAB with COBRA Toolbox V3, add the viral biomass objective function (VBOF, BioModels MODEL2003020001, Renz et al., 2020) to build infected generic models, then run FASTCC to remove flux-inconsistent reactions. Feed the consistent generic models plus discretized RNA-seq data (FPKM/TPM) into rFASTCORMICS (function fastcormics_RNAseq, consensus default 0.9) to build context-specific mock and infected lung models; the DCcov study produced 50 models (28 infected, 22 mock) from GSE147507 and GSE148729. Run FBA to determine maximal host biomass flux, fix host biomass upper bound to 10% of that maximum, and optimize the viral biomass objective. Execute singleGeneDeletion (and double-gene deletion for pairs) across all genes, classify a gene as essential when the grRatio (post-knockout viral biomass / pre-knockout maximum) is below 0.2. Map identified essential genes to DrugBank V5 for single-drug and combination repositioning, excluding gene pairs targeted by the same drug.

## Related tools

- **rFASTCORMICS** (Reconstructs context-specific metabolic models from RNA-seq expression data using the FASTCORMICS algorithm family) — https://github.com/sysbiolux/rFASTCORMICS
- **COBRA Toolbox V3** (MATLAB toolbox for loading models, running FBA, and executing singleGeneDeletion knockout analyses) — https://github.com/opencobra/cobratoolbox
- **Recon2.04** (Generic genome-scale reconstruction augmented with the VBOF as input for context-specific reconstruction)
- **Recon3D_01** (Generic genome-scale reconstruction augmented with the VBOF as input for context-specific reconstruction and pathway (subSystems) annotation)
- **MATLAB** (Execution environment for rFASTCORMICS and COBRA Toolbox workflows)

## Examples

```
[model, A_final] = fastcormics_RNAseq(model, data, rownames, dico, biomass_rxn, already_mapped_tag, consensus_proportion, epsilon, optional_settings)
```

## Evaluation signals

- VBOF-augmented models carry a nonzero viral biomass flux and no blocked reactions remain after FASTCC
- Context-specific reconstruction yields 50 models (28 infected, 22 mock) within the reported size range (median 3646 metabolites, 2456.5 reactions)
- Single-gene deletion reproduces exactly 23 unique essential genes, including CRLS1, SGMS1, GUK1, DTYMK, CMPK1, and TYMS, at the grRatio < 0.2 essentiality threshold
- Host biomass upper bound is fixed to 10% of the pre-knockout maximum so host growth does not mask viral biomass changes
- Exported SBML model files load cleanly in the COBRA Toolbox

## Limitations

- The work is limited by the modeled lung cell lines (A549, Calu-3, H1299, NHBE)
- Identified drug and drug pairs are based on network effects on the host metabolome (the virus is only modeled through its biomass function), not direct docking on the viral proteome
- Further in vitro single- and double-gene deletion studies, IC-50/synergy experiments, and toxicity assessment are needed before considering predictions as drug candidates

## Evidence

- [abstract] host-specific essential genes and gene pairs were determined through in silico knockouts that permit reducing the viral biomass production without affecting the host biomass: "host-specific essential genes and gene pairs were determined through in silico knockouts that permit reducing the viral biomass production without affecting the host biomass"
- [intro] VBOF added to generic models to build infected generic models: "The viral biomass function (VBOF) was added to the generic metabolic models (Recon2.04 and Recon3D_01)"
- [results] Model reconstruction scale reported in results: "resulted in 50 models (28 infected and 22 mock) with a median of 3646 metabolites (2465-5088) and 2456,5 reactions (1790-3474)"
- [readme] rFASTCORMICS README description of input data and algorithm: [evidence span withheld — anchor status: external_doc]
- [discussion] Discussion scope limitation on modeled cell lines: "the work is limited by the modeled lung cell lines (A549, Calu-3, H1299, NHBE)"
