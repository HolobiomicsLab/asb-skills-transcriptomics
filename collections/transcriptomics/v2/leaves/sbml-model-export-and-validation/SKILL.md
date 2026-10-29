---
name: sbml-model-export-and-validation
description: Use when a generic genome-scale reconstruction (e.g. Recon2.04 or Recon3D_01)
  must be made flux-consistent and exported as SBML to serve as the COBRA-format input
  for context-specific reconstruction with rFASTCORMICS.
when_to_use_negative:
- The model is already a context-specific model produced by rFASTCORMICS — this skill
  operates on the upstream generic reconstructions, not their outputs.
- The goal is reconstructing context-specific models from expression data — use the
  rFASTCORMICS skill with the exported consistent SBML; this skill only prepares and
  validates that input.
- The reconstruction is already consistent and no VBOF must be added — direct model
  loading/export without the FASTCC and identifier-harmonization steps suffices.
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0602
tools:
- name: COBRA Toolbox V3
  role: MATLAB environment in which Recon2.04/Recon3D_01 are loaded, VBOF is added,
    FASTCC is run, and SBML export/reload validation is performed
  repo: github.com/opencobra/cobratoolbox
- name: Recon2.04
  role: Generic human reconstruction made consistent (mock and VBOF-infected versions)
    and exported as SBML
- name: Recon3D_01
  role: Generic genome-scale reconstruction made consistent (mock and VBOF-infected
    versions) and exported as SBML
- name: rFASTCORMICS
  role: Downstream workflow whose COBRA-format input requirement defines the validation
    criteria for the exported SBML models
  repo: https://github.com/sysbiolux/rFASTCORMICS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:1c23386189879e9f6ae8185f6b86b542ceb9585246465a9d432e7a2804ab6f71
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# sbml-model-export-and-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepares VBOF-augmented generic reconstructions (Recon2.04, Recon3D_01) in MATLAB/COBRA Toolbox V3, prunes flux-inconsistent reactions with FASTCC, and exports the consistent infected and control models as SBML files that satisfy rFASTCORMICS' COBRA-format input requirements.

## When to use

Use when a generic genome-scale reconstruction (e.g. Recon2.04 or Recon3D_01) must be made flux-consistent and exported as SBML to serve as the COBRA-format input for context-specific reconstruction with rFASTCORMICS.

## When NOT to use

- The model is already a context-specific model produced by rFASTCORMICS — this skill operates on the upstream generic reconstructions, not their outputs.
- The goal is reconstructing context-specific models from expression data — use the rFASTCORMICS skill with the exported consistent SBML; this skill only prepares and validates that input.
- The reconstruction is already consistent and no VBOF must be added — direct model loading/export without the FASTCC and identifier-harmonization steps suffices.

## Inputs

- Generic genome-scale reconstructions Recon2.04 and Recon3D_01 from VMH (MAT/SBML)
- Viral biomass objective function (VBOF) from BioModels MODEL2003020001 (Renz et al., 2020)
- Metabolite identifier mapping between VBOF namespace and BiGG/VMH identifiers

## Outputs

- Consistent infected generic model (VBOF-augmented) exported as SBML
- Consistent control (mock) generic model exported as SBML, ready as rFASTCORMICS input

## How to apply

Load Recon2.04 or Recon3D_01 (VMH version) in MATLAB with COBRA Toolbox V3, then add the viral biomass objective function (VBOF, BioModels MODEL2003020001, Renz et al., 2020) as an extra biomass reaction, harmonizing metabolite identifiers between the VBOF namespace and the BiGG/VMH identifiers of the reconstruction. Keep the original (mock) reconstruction as the control generic model alongside the VBOF-augmented infected model. Run FASTCC on each to remove blocked (flux-inconsistent) reactions, which is required because rFASTCORMICS explicitly expects a '(consistent) genome-scale metabolic reconstruction in the COBRA format'. Export both the consistent infected and consistent control models as SBML files. Validate by reloading each SBML in the COBRA Toolbox and confirming the VBOF-augmented model carries a nonzero viral biomass flux with no remaining blocked reactions.

## Related tools

- **COBRA Toolbox V3** (MATLAB environment in which Recon2.04/Recon3D_01 are loaded, VBOF is added, FASTCC is run, and SBML export/reload validation is performed) — github.com/opencobra/cobratoolbox
- **Recon2.04** (Generic human reconstruction made consistent (mock and VBOF-infected versions) and exported as SBML)
- **Recon3D_01** (Generic genome-scale reconstruction made consistent (mock and VBOF-infected versions) and exported as SBML)
- **rFASTCORMICS** (Downstream workflow whose COBRA-format input requirement defines the validation criteria for the exported SBML models) — https://github.com/sysbiolux/rFASTCORMICS

## Evaluation signals

- Each exported SBML file reloads without error in the COBRA Toolbox (MATLAB).
- VBOF-augmented (infected) models carry a nonzero viral biomass flux under FBA.
- No blocked (flux-inconsistent) reactions remain after the FASTCC pruning step.
- VBOF metabolite identifiers are correctly mapped to the reconstruction's BiGG/VMH namespace before export.
- The exported model is accepted as the COBRA-format 'model' input of fastcormics_RNAseq (e.g. Recon 2.04 from VMH downloads).

## Limitations

- The VBOF uses a different metabolite namespace than the reconstructions, so identifier harmonization is a manual, error-prone step that must precede export.
- Consistency is achieved only via FASTCC pruning of blocked reactions; reactions removed by FASTCC are permanently absent from the exported SBML.
- The exported models are generic (not context-specific); rFASTCORMICS further requires the biomass reaction name (e.g. biomass_reaction, biomass_human) so the biomass can always carry flux, and only accepts consistently pruned COBRA-format models.

## Evidence

- [readme] rFASTCORMICS requires a consistent COBRA-format model as input: [evidence span withheld — anchor status: external_doc]
- [readme] Biomass reaction must be enabled to carry flux in the model: [evidence span withheld — anchor status: external_doc]
