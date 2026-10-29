---
name: synthetic-lethality-pair-identification
description: Use when you have context-specific metabolic models with a viral biomass
  objective function and need to enumerate gene pairs whose joint knockout eliminates
  viral biomass while single knockouts remain viable.
when_to_use_negative:
- When only single gene essentiality is needed — run singleGeneDeletion instead
- When the model does not carry a viral biomass objective function — synthetic-lethal
  pairs specific to viral replication cannot be scored
- When testing drug combinations directly rather than gene knockouts — use the downstream
  drug repositioning workflow (DrugBank V5 mapping)
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: COBRA Toolbox V3
  role: Provides the doubleGeneDeletion routine for systematic pairwise gene knockout
    on the infected metabolic models
  repo: https://github.com/opencobra/cobratoolbox
- name: MATLAB
  role: Runtime environment for executing COBRA Toolbox routines and managing model
    objects
- name: rFASTCORMICS
  role: Produces the infected context-specific models that serve as input to double
    gene deletion
  repo: https://github.com/sysbiolux/rFASTCORMICS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:01604b3e92adba8d8c0a509be9a0522207e1c6011fb48b26bfcea0879729fac2
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# synthetic-lethality-pair-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies synthetic-lethal (synergistic) gene pairs via in silico double gene deletion (doubleGeneDeletion) on context-specific infected metabolic models carrying a viral biomass objective function (VBOF), producing a deduplicated set of essential gene pairs that abolish viral biomass without affecting host biomass.

## When to use

Use when you have context-specific metabolic models with a viral biomass objective function and need to enumerate gene pairs whose joint knockout eliminates viral biomass while single knockouts remain viable.

## When NOT to use

- When only single gene essentiality is needed — run singleGeneDeletion instead
- When the model does not carry a viral biomass objective function — synthetic-lethal pairs specific to viral replication cannot be scored
- When testing drug combinations directly rather than gene knockouts — use the downstream drug repositioning workflow (DrugBank V5 mapping)

## Inputs

- Infected context-specific genome-scale metabolic models in COBRA format (MATLAB struct with VBOF) reconstructed from RNA-seq data via rFASTCORMICS
- Viral biomass objective function (VBOF) appended to the model
- GSE147507 (severity study) and GSE148729 (time-series study) expression data used for model reconstruction

## Outputs

- essential_gene_pairs.csv containing 598 unique essential gene pairs
- Classification of each pair as non-essential synergistic or essential-plus-non-essential
- Viral biomass production flux score per knockout pair

## How to apply

Load the infected context-specific genome-scale metabolic models (built by rFASTCORMICS from GSE147507 severity and GSE148729 time-series RNA-seq data, carrying the VBOF) into MATLAB with COBRA Toolbox V3. Run the doubleGeneDeletion routine on each infected model to systematically knock out all gene pairs and score each by the resulting viral biomass production flux. Classify knockout results into non-essential synergistic pairs (each single knockout viable, double knockout abolishes viral biomass) and essential-plus-non-essential pairs (one essential gene paired with a non-essential gene). Concatenate the two pair classes across all infected models and deduplicate to obtain the unique essential gene pair set; the reference implementation yielded 598 unique essential gene pairs, of which 56 are druggable. Validate by confirming exactly 598 unique gene pairs and that each listed pair reduces viral biomass production without affecting host biomass maintenance.

## Related tools

- **COBRA Toolbox V3** (Provides the doubleGeneDeletion routine for systematic pairwise gene knockout on the infected metabolic models) — https://github.com/opencobra/cobratoolbox
- **MATLAB** (Runtime environment for executing COBRA Toolbox routines and managing model objects)
- **rFASTCORMICS** (Produces the infected context-specific models that serve as input to double gene deletion) — https://github.com/sysbiolux/rFASTCORMICS

## Examples

```
[gr] = doubleGeneDeletion(infected_model);
```

## Evaluation signals

- Output contains exactly 598 unique gene pairs after deduplication across all infected models
- Each listed gene pair reduces viral biomass production flux compared to the wild-type infected model
- Each listed gene pair does not affect host biomass maintenance (host biomass flux remains above viability threshold)
- For synergistic pairs specifically, each single gene knockout remains viable (nonzero viral biomass)
- 56 of the 598 unique pairs map to druggable targets when cross-referenced with DrugBank V5

## Limitations

- Predictions require further in vitro single- and double-gene deletion studies to confirm essentiality of the identified gene pairs
- The models are limited by the modeled lung cell lines (A549, Calu-3, H1299, NHBE)
- Essentiality is scored against viral biomass only; drug targets reflect network effects on the host metabolome, not direct docking on the viral proteome

## Evidence
