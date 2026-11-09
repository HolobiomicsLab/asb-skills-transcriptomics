---
name: constraint-based-flux-balance-analysis
description: Use when you have expression data from infected vs. mock tissue (e.g.
  GSE147507, GSE148729) and need to infer metabolic gene essentiality or drug targets
  via constraint-based modeling of an infected host model.
when_to_use_negative:
- The virus must be modeled as an intracellular replicant with a defined VBOF; the
  drug predictions here rely on network effects on the host metabolome, not direct
  docking on the viral proteome, so this skill is wrong for virion-assembly or protein-targeting
  questions
- Only lung cell lines represented in the datasets (A549, Calu-3, H1299, NHBE) were
  modeled; applying to unmodeled tissues requires additional validation and other
  optimization functions
- If no viral biomass objective function is available for the virus of interest, essentiality
  of viral-specific genes cannot be assessed this way
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: MATLAB
  role: Runtime environment in which the COBRA Toolbox V3 knockout and FBA routines
    are executed
- name: COBRA Toolbox V3
  role: Provides the FBA and doubleGeneDeletion routines used to systematically knock
    out gene pairs and score viral biomass production flux
  repo: https://github.com/opencobra/cobratoolbox/tree/master/src
- name: rFASTCORMICS
  role: Reconstructs the context-specific mock and infected lung metabolic models
    from RNA-seq expression data as input to the FBA analysis
  repo: https://github.com/sysbiolux/rFASTCORMICS
- name: Recon3D_01
  role: Generic genome-scale reconstruction carrying the VBOF from which the infected
    context-specific models are derived
- name: DESeq2
  role: Computes differentially expressed genes (absolute log2 fold change >1, adjusted
    p-value < 0.05) used to constrain the context-specific models
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:d6c75255778de7700aa05a67d22673cbe81bc70f28fb52a9e94bbe0199bc3ccc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sysbiolux/DCcov
  tools:
  - COBRA Toolbox
  - COBRA Toolbox V3
  - DESeq2
  - MATLAB
  - Recon3D_01
  - rFASTCORMICS
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# constraint-based-flux-balance-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply flux balance analysis (FBA) on context-specific genome-scale metabolic models of SARS-CoV-2-infected lung cells (built with rFASTCORMICS and carrying a viral biomass objective function, VBOF) to identify host essential genes and gene pairs whose in silico knockout reduces viral biomass without affecting host biomass.

## When to use

Use when you have expression data from infected vs. mock tissue (e.g. GSE147507, GSE148729) and need to infer metabolic gene essentiality or drug targets via constraint-based modeling of an infected host model.

## When NOT to use

- The virus must be modeled as an intracellular replicant with a defined VBOF; the drug predictions here rely on network effects on the host metabolome, not direct docking on the viral proteome, so this skill is wrong for virion-assembly or protein-targeting questions
- Only lung cell lines represented in the datasets (A549, Calu-3, H1299, NHBE) were modeled; applying to unmodeled tissues requires additional validation and other optimization functions
- If no viral biomass objective function is available for the virus of interest, essentiality of viral-specific genes cannot be assessed this way

## Inputs

- RNA-seq expression data from infected and mock lung samples (FPKM/TPKM-transformed), e.g. GEO: GSE147507 (severity study) and GSE148729 (time-series study)
- Consistent genome-scale metabolic reconstruction in COBRA format (Recon2.04 or Recon3D_01)
- Gene identifier mapping table (dico) from Biomart
- Viral biomass objective function (VBOF) from Renz et al., 2020
- Differentially expressed genes (DESeq2: absolute log2 fold change >1, adjusted p-value < 0.05)

## Outputs

- Context-specific mock and infected genome-scale metabolic models (50 models: 28 infected, 22 mock)
- List of essential genes and unique essential gene pairs (e.g. essential_gene_pairs.csv with 598 unique pairs)
- Knockout essentiality scores based on viral biomass production flux
- Druggable essential gene/gene-pair set mapped to DrugBank V5 (56 druggable pairs)

## How to apply

Load a consistent genome-scale reconstruction (Recon2.04 or Recon3D_01) in MATLAB with COBRA Toolbox V3, add the viral biomass objective function (VBOF), then build context-specific mock and infected models from RNA-seq data using rFASTCORMICS with discretized expression input (1 expressed, -1 not expressed, 0 unknown). Run in silico single and double gene knockouts (doubleGeneDeletion) on each infected model, scoring each knockout by viral biomass production flux. Classify results as essential genes, non-essential synergistic pairs (both single knockouts viable, double knockout abolishes viral biomass), and essential-plus-non-essential pairs, then deduplicate across all infected models. Validate by confirming each hit reduces viral biomass while leaving human biomass maintenance unaffected, then map druggable targets to DrugBank V5 for repositioning.

## Related tools

- **MATLAB** (Runtime environment in which the COBRA Toolbox V3 knockout and FBA routines are executed)
- **COBRA Toolbox V3** (Provides the FBA and doubleGeneDeletion routines used to systematically knock out gene pairs and score viral biomass production flux) — https://github.com/opencobra/cobratoolbox/tree/master/src
- **rFASTCORMICS** (Reconstructs the context-specific mock and infected lung metabolic models from RNA-seq expression data as input to the FBA analysis) — https://github.com/sysbiolux/rFASTCORMICS
- **Recon3D_01** (Generic genome-scale reconstruction carrying the VBOF from which the infected context-specific models are derived)
- **DESeq2** (Computes differentially expressed genes (absolute log2 fold change >1, adjusted p-value < 0.05) used to constrain the context-specific models)

## Examples

```
[model, A_final] = fastcormics_RNAseq(model, data, rownames, dico, biomass_rxn, already_mapped_tag, consensus_proportion, epsilon, optional_settings)
```

## Evaluation signals

- Each reported essential gene or gene pair must reduce viral biomass production flux while leaving host biomass maintenance unchanged
- The deduplicated gene-pair output must match the expected unique-pair count (598 pairs across the severity and time-series studies, of which 56 are druggable)
- Reconstructed models should fall in the reported size range: 50 models (28 infected, 22 mock), median 3646 metabolites (2465-5088) and 2456.5 reactions (1790-3474)
- Single-knockout viability check: pairs counted as synergistic must have both single knockouts viable with only the double knockout abolishing viral biomass
- Model sanity check: the VBOF must carry flux in every infected model used for knockout screening

## Limitations

- Predicted drugs and drug pairs are based on network effects on the host metabolome (the virus is only modeled through its biomass function) rather than direct docking on the viral proteome
- The work is limited by the modeled lung cell lines (A549, Calu-3, H1299, NHBE)
- No drug response curves, IC-50, or synergy experiments were performed; further in vitro single- and double-gene deletion studies are needed to confirm essentiality, and toxicity of predicted drugs requires careful assessment

## Evidence

- [intro] FBA on infected lung models to find viral-specific essential targets: "we employ here FBA to find candidate drugs and drug combinations that target viral-specific essential genes in SARS-CoV-2-infected lung cells"
- [abstract] FBA described as an efficient metabolic network method: "Flux balance analysis (FBA) is an efficient method to analyze metabolic networks."
- [readme] COBRA Toolbox as the constraint-based modeling suite: [evidence span withheld — anchor status: external_doc]
- [results] Scale of reconstructed context-specific models: "resulted in 50 models (28 infected and 22 mock) with a median of 3646 metabolites (2465-5088) and 2456,5 reactions (1790-3474)"
- [readme] rFASTCORMICS builds the context-specific models from RNA-seq data: [evidence span withheld — anchor status: external_doc]
