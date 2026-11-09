---
name: viral-biomass-objective-integration
description: Use when building infected-host metabolic models that must carry a viral
  biomass reaction (e.g. for FBA-based viral essentiality or drug repositioning),
  starting from a generic reconstruction plus a published VBOF.
when_to_use_negative:
- When the goal is to model the virus through direct docking or viral-proteome network
  effects — here the virus is represented only through its biomass function on the
  host model
- When the reconstruction is already flux-consistent and already carries the VBOF;
  re-running identifier harmonization and FASTCC is redundant
- For non-viral context-specific reconstructions where a host biomass function alone
  suffices (plain rFASTCORMICS with a host biomass reaction is the correct path)
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: Recon2.04
  role: Generic genome-scale reconstruction to which the VBOF is added to build the
    infected generic model
- name: Recon3D_01
  role: Second generic genome-scale reconstruction augmented with the VBOF; also the
    reference for subSystems mapping downstream
- name: MATLAB
  role: Environment in which Recon2.04/Recon3D_01 are loaded and the VBOF is integrated
- name: COBRA Toolbox V3
  role: Provides model loading, FASTCC (blocked-reaction removal), and SBML export/import
    used to build the consistent generic models
  repo: https://github.com/opencobra/cobratoolbox/tree/master/src
- name: rFASTCORMICS
  role: Downstream consumer of the consistent infected/control SBML models as its
    reconstruction base
  repo: https://github.com/sysbiolux/rFASTCORMICS
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:9dbd8e312667375f6d762ddcc85fa71ac3268ae8cf81ded3c61e0309ab034d86
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
  - MATLAB
  - Recon2.04
  - Recon3D_01
  - rFASTCORMICS
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# viral-biomass-objective-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Augments a generic genome-scale metabolic reconstruction (Recon2.04, Recon3D_01) with the SARS-CoV-2 viral biomass objective function (VBOF, BioModels MODEL2003020001) and removes flux-inconsistent reactions with FASTCC, yielding consistent infected and control generic models that serve as inputs for context-specific (rFASTCORMICS) reconstruction and viral-essentiality analysis.

## When to use

Use when building infected-host metabolic models that must carry a viral biomass reaction (e.g. for FBA-based viral essentiality or drug repositioning), starting from a generic reconstruction plus a published VBOF.

## When NOT to use

- When the goal is to model the virus through direct docking or viral-proteome network effects — here the virus is represented only through its biomass function on the host model
- When the reconstruction is already flux-consistent and already carries the VBOF; re-running identifier harmonization and FASTCC is redundant
- For non-viral context-specific reconstructions where a host biomass function alone suffices (plain rFASTCORMICS with a host biomass reaction is the correct path)

## Inputs

- Genome-scale metabolic reconstruction Recon2.04 (VMH, COBRA/SBML format)
- Genome-scale metabolic reconstruction Recon3D_01 (VMH, COBRA/SBML format)
- Viral biomass objective function (VBOF) from BioModels MODEL2003020001 (Renz et al., 2020)
- Metabolite identifier mapping between the VBOF namespace and BiGG/VMH identifiers

## Outputs

- Consistent infected (VBOF-added) generic models Recon2.04 and Recon3D_01 as SBML files
- Consistent control (mock) generic models Recon2.04 and Recon3D_01 as SBML files
- SBML files validated to load in the COBRA Toolbox with nonzero viral biomass flux and no blocked reactions

## How to apply

Load the generic reconstructions Recon2.04 and Recon3D_01 (VMH downloads, COBRA format) in MATLAB with the COBRA Toolbox V3. Retrieve the viral biomass objective function (VBOF) from BioModels (MODEL2003020001, Renz et al., 2020) and harmonize metabolite identifiers between the VBOF namespace and the BiGG/VMH identifiers of each reconstruction before adding it as an extra biomass reaction — this yields the infected generic models, while the original (mock) versions are kept as control generic models. Run FASTCC on the VBOF-augmented models to remove blocked (flux-inconsistent) reactions, then export the consistent infected and control generic models as SBML files for the rFASTCORMICS context-specific workflow. Validation relies on each output SBML loading in the COBRA Toolbox and on VBOF-augmented models carrying a nonzero viral biomass flux with no blocked reactions remaining after FASTCC.

## Related tools

- **Recon2.04** (Generic genome-scale reconstruction to which the VBOF is added to build the infected generic model)
- **Recon3D_01** (Second generic genome-scale reconstruction augmented with the VBOF; also the reference for subSystems mapping downstream)
- **MATLAB** (Environment in which Recon2.04/Recon3D_01 are loaded and the VBOF is integrated)
- **COBRA Toolbox V3** (Provides model loading, FASTCC (blocked-reaction removal), and SBML export/import used to build the consistent generic models) — https://github.com/opencobra/cobratoolbox/tree/master/src
- **rFASTCORMICS** (Downstream consumer of the consistent infected/control SBML models as its reconstruction base) — https://github.com/sysbiolux/rFASTCORMICS

## Examples

```
model = readCbModel('Recon204_VBOF_added.mat'); consistent = fastcc(model); writeCbModel(consistent, 'sbml', 'consistent_Recon204_VBOF.xml')
```

## Evaluation signals

- Each exported SBML model loads in the COBRA Toolbox without errors
- VBOF-augmented (infected) models carry a nonzero viral biomass flux under FBA
- No blocked (flux-inconsistent) reactions remain after running FASTCC
- The consistent control (mock) model retains the host biomass reaction and no VBOF
- Downstream context-specific reconstruction proceeds without missing-reaction errors when the consistent models are supplied to rFASTCORMICS

## Limitations

- The virus is modeled only through its biomass function, so predictions reflect network effects on the host metabolome rather than direct antiviral/proteome effects
- Results depend on the modeled host; this work was limited to lung cell lines (A549, Calu-3, H1299, NHBE) and further tests on other tissues and optimization functions were flagged as required
- Identifier harmonization between the VBOF namespace and BiGG/VMH identifiers must be done manually and can silently drop VBOF metabolites if mapping is incomplete

## Evidence

- [readme] consistent model prerequisite in rFASTCORMICS README: [evidence span withheld — anchor status: external_doc]
