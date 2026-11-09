---
name: viral-biomass-objective-knockout-scoring
description: Use when you have infected context-specific genome-scale metabolic models
  with a VBOF (e.g. SARS-CoV-2 lung models) and need to enumerate essential genes
  or synergistic gene pairs as drug-target candidates.
when_to_use_negative:
- The model does not carry a viral biomass objective function — knockouts then only
  probe host biomass, not virus-specific essentiality
- The goal is direct docking or antiviral targeting of the viral proteome; this skill
  identifies host metabolic targets via network effects on the host metabolome only
- You need experimentally validated essentiality rather than in silico predictions
  — in vitro single- and double-gene deletion studies are required to confirm the
  predictions
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
tools:
- name: COBRA Toolbox V3
  role: Provides the doubleGeneDeletion routine used to systematically knock out gene
    pairs and score viral biomass flux in MATLAB
  repo: https://github.com/opencobra/cobratoolbox
- name: MATLAB
  role: Runtime environment in which the infected models are loaded and knockout analyses
    are executed
- name: rFASTCORMICS
  role: Upstream reconstruction of the infected context-specific models that the knockouts
    are run on
  repo: https://github.com/sysbiolux/rFASTCORMICS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:27d19786d2758711492f079774fd5d9e131f8a8d94401899bd15622ba157e72f
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
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Viral biomass objective knockout scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Systematically performs in silico single- and double-gene knockouts on context-specific metabolic models carrying a viral biomass objective function (VBOF), scoring each knockout by the resulting viral biomass production flux to identify host genes whose loss reduces viral biomass without affecting host biomass maintenance.

## When to use

Use when you have infected context-specific genome-scale metabolic models with a VBOF (e.g. SARS-CoV-2 lung models) and need to enumerate essential genes or synergistic gene pairs as drug-target candidates.

## When NOT to use

- The model does not carry a viral biomass objective function — knockouts then only probe host biomass, not virus-specific essentiality
- The goal is direct docking or antiviral targeting of the viral proteome; this skill identifies host metabolic targets via network effects on the host metabolome only
- You need experimentally validated essentiality rather than in silico predictions — in vitro single- and double-gene deletion studies are required to confirm the predictions

## Inputs

- Infected context-specific genome-scale metabolic models in COBRA format (e.g. Recon3D_01-based lung models for A549, Calu-3, H1299, NHBE) with a viral biomass objective function (VBOF)
- Host biomass maintenance reaction definition
- Consistent generic reconstruction (Recon2.04 or Recon3D_01) and medium constraints used for the reconstruction

## Outputs

- List of unique essential gene pairs (598 in the source study, e.g. essential_gene_pairs.csv), classified into non-essential synergistic pairs and essential-plus-non-essential pairs
- Per-knockout viral biomass production flux scores
- Essential single genes (e.g. GUK1) whose knockout decreases viral biomass without affecting human biomass maintenance

## How to apply

Load the infected context-specific models (reconstructed with rFASTCORMICS from datasets such as GSE147507 and GSE148729, carrying the VBOF from Renz et al. 2020 on Recon2.04/Recon3D_01) into MATLAB with the COBRA Toolbox V3. Run the doubleGeneDeletion routine on each infected model to knock out all gene pairs, scoring each pair by the resulting viral biomass production flux. Classify results into non-essential synergistic pairs (each single knockout viable, the double knockout abolishes viral biomass) and essential-plus-non-essential pairs; a knockout is only accepted as essential if it permits reducing viral biomass production without affecting the host biomass. Concatenate pair classes across all infected models and deduplicate — in the DCcov study this yielded 598 unique essential gene pairs across the severity and time-series studies, of which 56 were druggable. Export the deduplicated pairs (e.g. essential_gene_pairs.csv) and map them to DrugBank V5 for single-drug and combination repositioning, excluding pairs where both genes are targeted by the same drug.

## Related tools

- **COBRA Toolbox V3** (Provides the doubleGeneDeletion routine used to systematically knock out gene pairs and score viral biomass flux in MATLAB) — https://github.com/opencobra/cobratoolbox
- **MATLAB** (Runtime environment in which the infected models are loaded and knockout analyses are executed)
- **rFASTCORMICS** (Upstream reconstruction of the infected context-specific models that the knockouts are run on) — https://github.com/sysbiolux/rFASTCORMICS

## Examples

```
results = doubleGeneDeletion(infectedModel); % COBRA Toolbox V3 in MATLAB: knock out all gene pairs, score by viral biomass flux
```

## Evaluation signals

- The deduplicated output contains exactly 598 unique essential gene pairs across the severity and time-series infected models
- Each listed pair reduces viral biomass production while leaving host biomass maintenance unaffected
- Knockout results split cleanly into non-essential synergistic pairs (both single knockouts viable) and essential-plus-non-essential pairs
- Druggability mapping identifies 56 druggable gene pairs among the essential pairs (eight genes druggable by 45 unique drugs for single targets)
- Sanity check on reconstruction inputs: infected models should derive from the 28 infected (of 50 total) rFASTCORMICS models with a VBOF added

## Limitations

- Essentiality predictions are in silico only; further in vitro single- and double-gene deletion studies are needed to confirm them
- Drug targets are based on network effects on the host metabolome (the virus is modeled only through its biomass function), not direct docking on the viral proteome
- Predictions are limited to the modeled lung cell lines (A549, Calu-3, H1299, NHBE) and would require testing on other tissues and other optimization functions plus in vitro/in vivo validation

## Evidence

- [methods] Run the doubleGeneDeletion routine on each infected model to systematically knock out all gene pairs and score each pair by the resulting viral biomass production flux.: [evidence span withheld — anchor status: not_found]
- [readme] - Matlab 2013 or higher
		- compatible IBM ILOG CPLEX installation: "- Matlab 2013 or higher
		- compatible IBM ILOG CPLEX installation"
