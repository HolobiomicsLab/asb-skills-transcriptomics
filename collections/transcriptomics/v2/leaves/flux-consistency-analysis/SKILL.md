---
name: flux-consistency-analysis
description: Use when a generic genome-scale reconstruction (e.g., Recon2.04, Recon3D_01
  from VMH) — possibly augmented with a custom biomass reaction such as a viral biomass
  objective function (VBOF) — must be made flux-consistent before context-specific
  reconstruction.
when_to_use_negative:
- The model is already flux-consistent (e.g., a published consistent reconstruction)
  — re-running FASTCC is redundant and may remove reactions unnecessarily.
- The goal is a fully unconstrained research model rather than an input to FASTCORMICS-family
  context-specific reconstruction, where removing reactions could alter the research
  network.
- The input is gene expression data rather than a model — consistency pruning operates
  on the reconstruction, not on expression data (expression enters later via rFASTCORMICS).
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: COBRA Toolbox V3
  role: Provides the FASTCC flux-consistency algorithm and SBML import/export used
    to prune blocked reactions from Recon2.04 and Recon3D_01
  repo: https://github.com/opencobra/cobratoolbox
- name: Recon2.04
  role: Generic genome-scale reconstruction made consistent (with and without the
    VBOF) by this skill
- name: Recon3D_01
  role: Generic genome-scale reconstruction made consistent (with and without the
    VBOF) by this skill
- name: rFASTCORMICS
  role: Downstream consumer requiring the consistent generic models as input, motivating
    the pruning step
  repo: https://github.com/sysbiolux/rFASTCORMICS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:6d3019ce7841bcaaf937c6123cebc1ba36eb7d9163bc59486aca80832c97d2d6
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# flux-consistency-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prunes blocked (flux-inconsistent) reactions from a genome-scale metabolic model using FASTCC in the COBRA Toolbox, as done in DCcov where VBOF-augmented Recon2.04 and Recon3D_01 infected generic models and their mock controls were converted into consistent inputs for rFASTCORMICS.

## When to use

Use when a generic genome-scale reconstruction (e.g., Recon2.04, Recon3D_01 from VMH) — possibly augmented with a custom biomass reaction such as a viral biomass objective function (VBOF) — must be made flux-consistent before context-specific reconstruction.

## When NOT to use

- The model is already flux-consistent (e.g., a published consistent reconstruction) — re-running FASTCC is redundant and may remove reactions unnecessarily.
- The goal is a fully unconstrained research model rather than an input to FASTCORMICS-family context-specific reconstruction, where removing reactions could alter the research network.
- The input is gene expression data rather than a model — consistency pruning operates on the reconstruction, not on expression data (expression enters later via rFASTCORMICS).

## Inputs

- Genome-scale metabolic reconstruction in COBRA/SBML format (Recon2.04 or Recon3D_01 from VMH)
- Viral biomass objective function (VBOF) from BioModels MODEL2003020001 (Renz et al., 2020), for infected-model variants
- MATLAB environment with COBRA Toolbox V3

## Outputs

- Consistent infected generic metabolic model (VBOF-augmented, blocked reactions removed), exported as SBML
- Consistent control (mock) generic metabolic model, exported as SBML
- SBML files validated to load in the COBRA Toolbox as rFASTCORMICS inputs

## How to apply

Load the reconstruction (Recon2.04 or Recon3D_01 in VMH/BiGG namespace) in MATLAB with the COBRA Toolbox V3. For infected models, first retrieve the viral biomass objective function (VBOF) from BioModels (MODEL2003020001, Renz et al., 2020), harmonize its metabolite identifiers to each reconstruction's namespace, and add it as an extra biomass reaction; keep the original reconstruction as the mock/control version. Then run FASTCC on each model to remove blocked reactions, producing a consistent infected (VBOF-added) and a consistent control generic model. Export both as SBML files, since rFASTCORMICS requires a '(consistent) genome-scale metabolic reconstruction in the COBRA format' as its model input. The rationale is that FASTCORMICS-family algorithms assume a consistent reference model, so removing flux-inconsistent reactions prevents artefactual gaps in the downstream context-specific models.

## Related tools

- **COBRA Toolbox V3** (Provides the FASTCC flux-consistency algorithm and SBML import/export used to prune blocked reactions from Recon2.04 and Recon3D_01) — https://github.com/opencobra/cobratoolbox
- **Recon2.04** (Generic genome-scale reconstruction made consistent (with and without the VBOF) by this skill)
- **Recon3D_01** (Generic genome-scale reconstruction made consistent (with and without the VBOF) by this skill)
- **rFASTCORMICS** (Downstream consumer requiring the consistent generic models as input, motivating the pruning step) — https://github.com/sysbiolux/rFASTCORMICS

## Examples

```
modelInfected = addVBOF(loadDistributedModel('RECON2.04')); modelConsistent = fastcc(modelInfected); writeSBML(modelConsistent, 'Recon2.04_infected_consistent.xml');
```

## Evaluation signals

- Each exported consistent model file loads in the COBRA Toolbox without errors.
- VBOF-augmented models carry a nonzero viral biomass flux at the optimal solution.
- No blocked reactions remain after FASTCC (i.e., every reaction can carry flux in some steady-state distribution).
- The consistent control model corresponds to the original (mock) Recon2.04 or Recon3D_01 with only blocked reactions removed.
- Both consistent infected and control models are accepted as valid rFASTCORMICS 'model' inputs.

## Limitations

- Consistency pruning discards flux-inconsistent reactions, so downstream models cannot recover those reactions even if expression data suggest activity.
- The VBOF must have its metabolite identifiers harmonized to each reconstruction's (BiGG/VMH) namespace before addition; mismatches will break the biomass flux test.
- The infected variant only approximates SARS-CoV-2 infection through the viral biomass function; the virus is otherwise not modeled.

## Evidence

- [readme] rFASTCORMICS requires a consistent model input: [evidence span withheld — anchor status: external_doc]
