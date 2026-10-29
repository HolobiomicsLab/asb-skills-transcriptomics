---
name: metabolite-identifier-harmonization-across-reconstructions
description: Use when an external biomass objective function (e.g. the VBOF from Renz
  et al., 2020) must be added to a generic reconstruction such as Recon2.04 or Recon3D_01
  whose metabolite identifiers use a different namespace, before running FASTCC and
  exporting consistent infected/control SBML models.
when_to_use_negative:
- The VBOF and the target reconstruction already share the same metabolite identifier
  namespace, so no cross-namespace mapping is needed
- The goal is building context-specific (sample-level) models from RNA-seq/expression
  data — that is the rFASTCORMICS step that consumes the outputs of this skill, not
  this skill itself
- The target model is not a consistent COBRA-format genome-scale reconstruction (e.g.
  a small cell-line model or non-metabolic network), since the workflow is defined
  for Recon2.04/Recon3D_01-style inputs
edam_operation: http://edamontology.org/operation_0281
edam_topics:
- http://edamontology.org/topic_2250
tools:
- name: COBRA Toolbox V3
  role: Used in MATLAB to load Recon2.04/Recon3D_01, add the VBOF as an extra biomass
    reaction, run FASTCC to remove blocked reactions, and export SBML
  repo: https://github.com/opencobra/cobratoolbox
- name: Recon2.04
  role: Generic human metabolic reconstruction to which the VBOF is added with harmonized
    identifiers (control version kept as mock)
- name: Recon3D_01
  role: Second generic human metabolic reconstruction (VMH) augmented with the VBOF
    and made flux-consistent via FASTCC
- name: rFASTCORMICS
  role: Downstream workflow whose required (consistent) genome-scale model input is
    produced by this harmonization/augmentation step
  repo: https://github.com/sysbiolux/rFASTCORMICS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:9ee53252de03c28f78bc98e8023cb581de48a528c12dacf92a62e9b0f9e8e5b1
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
  - Recon2.04
  - Recon3D_01
  - rFASTCORMICS
  edam_operation: http://edamontology.org/operation_0281
  edam_topics:
  - http://edamontology.org/topic_2250
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# metabolite-identifier-harmonization-across-reconstructions

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

This skill merges an external biomass reaction (the SARS-CoV-2 viral biomass objective function, VBOF, from BioModels MODEL2003020001) into generic genome-scale reconstructions (Recon2.04, Recon3D_01) in MATLAB with the COBRA Toolbox V3, by reconciling the VBOF metabolite namespace with the BiGG/VMH identifiers of each reconstruction so the augmented models are consistent and usable as rFASTCORMICS inputs.

## When to use

Use when an external biomass objective function (e.g. the VBOF from Renz et al., 2020) must be added to a generic reconstruction such as Recon2.04 or Recon3D_01 whose metabolite identifiers use a different namespace, before running FASTCC and exporting consistent infected/control SBML models.

## When NOT to use

- The VBOF and the target reconstruction already share the same metabolite identifier namespace, so no cross-namespace mapping is needed
- The goal is building context-specific (sample-level) models from RNA-seq/expression data — that is the rFASTCORMICS step that consumes the outputs of this skill, not this skill itself
- The target model is not a consistent COBRA-format genome-scale reconstruction (e.g. a small cell-line model or non-metabolic network), since the workflow is defined for Recon2.04/Recon3D_01-style inputs

## Inputs

- Generic genome-scale reconstructions Recon2.04 and Recon3D_01 in COBRA/SBML format (from VMH, https://vmh.uni.lu/#downloadview)
- Viral biomass objective function (VBOF) from BioModels MODEL2003020001 (Renz et al., 2020)

## Outputs

- VBOF-augmented (infected) generic Recon2.04 and Recon3D_01 models as SBML files
- Consistent infected and control generic models with blocked reactions removed by FASTCC, exported as SBML for rFASTCORMICS

## How to apply

Load Recon2.04 and Recon3D_01 (from VMH) in MATLAB with the COBRA Toolbox V3, keeping the original (mock) versions as control generic models. Retrieve the viral biomass objective function (VBOF) from BioModels (MODEL2003020001, Renz et al., 2020) and add it as an extra biomass reaction to each generic model; while doing so, harmonize metabolite identifiers between the VBOF namespace and the BiGG/VMH identifiers of each reconstruction so that all VBOF metabolites resolve to existing model species. Run FASTCC on the VBOF-augmented models to remove blocked (flux-inconsistent) reactions, then export the consistent infected (VBOF-added) and control generic models as SBML files. These SBML models then serve as the (consistent) genome-scale inputs required by the rFASTCORMICS context-specific reconstruction workflow.

## Related tools

- **COBRA Toolbox V3** (Used in MATLAB to load Recon2.04/Recon3D_01, add the VBOF as an extra biomass reaction, run FASTCC to remove blocked reactions, and export SBML) — https://github.com/opencobra/cobratoolbox
- **Recon2.04** (Generic human metabolic reconstruction to which the VBOF is added with harmonized identifiers (control version kept as mock))
- **Recon3D_01** (Second generic human metabolic reconstruction (VMH) augmented with the VBOF and made flux-consistent via FASTCC)
- **rFASTCORMICS** (Downstream workflow whose required (consistent) genome-scale model input is produced by this harmonization/augmentation step) — https://github.com/sysbiolux/rFASTCORMICS

## Evaluation signals

- Each exported output model file loads in the COBRA Toolbox without errors
- VBOF-augmented (infected) models carry a nonzero viral biomass flux after augmentation
- No blocked (flux-inconsistent) reactions remain in the models after FASTCC
- Every VBOF metabolite maps to an existing BiGG/VMH metabolite in the target reconstruction (no unmatched/placeholder species in the biomass reaction)
- Both consistent infected and control (mock) SBML versions are produced for each reconstruction (Recon2.04 and Recon3D_01)

## Limitations

- Identifier harmonization must be performed between the VBOF namespace and the BiGG/VMH identifiers of each reconstruction; unmatched metabolites would invalidate the viral biomass flux
- The rFASTCORMICS workflow requires a (consistent) genome-scale model in COBRA format, so exporting non-consistent or non-COBRA-format models breaks the downstream step
- Validation in the source workflow is limited to model loading, nonzero viral biomass flux, and absence of blocked reactions; biological fidelity of the merged biomass reaction is not separately assessed

## Evidence

- [methods] harmonizing metabolite identifiers between the VBOF namespace and the BiGG/VMH identifiers of each reconstruction: [evidence span withheld — anchor status: not_found]
- [methods] Run FASTCC on the VBOF-augmented models to remove blocked (flux-inconsistent) reactions: [evidence span withheld — anchor status: not_found]
- [readme] model: (consistent) genome-scale metabolic reconstruction in the COBRA format, i.e. Recon 2.04 (from https://vmh.uni.lu/#downloadview ): [evidence span withheld — anchor status: external_doc]
