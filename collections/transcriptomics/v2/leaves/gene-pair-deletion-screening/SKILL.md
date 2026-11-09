---
name: gene-pair-deletion-screening
description: Use when you have mock and infected context-specific genome-scale metabolic
  models (e.g., built from GSE147507/GSE148729 lung expression data) and need to find
  knockouts that selectively impair viral biomass while sparing host biomass.
when_to_use_negative:
- You lack a viral biomass objective function or the models were not built with the
  infected/mock contrast — essentiality calls against viral biomass are undefined
- You need direct antiviral activity against viral proteins — this skill targets host-metabolome
  network effects only, since the virus is modeled solely through its biomass function
- Your expression data are microarray-based — use FASTCORMICS rather than rFASTCORMICS,
  which expects RNA-seq (preferably FPKM/TPKM) input
tools:
- name: COBRA Toolbox V3
  role: Runs the in silico single- and double-gene deletion simulations on the context-specific
    models in MATLAB
  repo: https://github.com/opencobra/cobratoolbox
- name: rFASTCORMICS
  role: Reconstructs the mock and infected context-specific genome-scale metabolic
    models that the deletion screen is run on
  repo: https://github.com/sysbiolux/rFASTCORMICS
- name: MATLAB
  role: Execution environment for the COBRA Toolbox deletion workflows
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:d292cbc9e805d11fc150aed71c6b392e46e41caff1ca53f0380611828f3e2792
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gene-pair-deletion-screening

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies host-specific essential single genes and gene pairs in SARS-CoV-2-infected lung models by running in silico single- and double-gene deletion simulations with the COBRA Toolbox on rFASTCORMICS context-specific models carrying a viral biomass objective function (VBOF). A knockout is essential when it reduces viral biomass production without affecting host biomass, enabling prioritization of safe antiviral drug targets.

## When to use

Use when you have mock and infected context-specific genome-scale metabolic models (e.g., built from GSE147507/GSE148729 lung expression data) and need to find knockouts that selectively impair viral biomass while sparing host biomass.

## When NOT to use

- You lack a viral biomass objective function or the models were not built with the infected/mock contrast — essentiality calls against viral biomass are undefined
- You need direct antiviral activity against viral proteins — this skill targets host-metabolome network effects only, since the virus is modeled solely through its biomass function
- Your expression data are microarray-based — use FASTCORMICS rather than rFASTCORMICS, which expects RNA-seq (preferably FPKM/TPKM) input

## Inputs

- 22 mock and 28 infected context-specific genome-scale metabolic models in COBRA/MATLAB format (rFASTCORMICS output from Recon2.04 or Recon3D_01 plus VBOF)
- Viral biomass objective function (VBOF) reaction definition
- Host (human) biomass reaction definition

## Outputs

- Essential single-gene list with essentiality scores (counts across infected models) and safety scores (counts across mock models)
- Essential gene-pair list with per-pair essentiality and safety scores
- Essentiality-vs-safety scatter plot (Figure 3-style) highlighting GUK1 and the CRLS1/SGMS1/SLC27A4 gene pairs

## How to apply

Load the 22 mock and 28 infected context-specific genome-scale metabolic models (rFASTCORMICS-built from Recon2.04/Recon3D_01 with the VBOF from Renz et al., 2020) into MATLAB with the COBRA Toolbox V3. Run in silico single-gene deletion (singleDeletion) and double-gene deletion simulations on both model sets, flagging a knockout as essential when viral biomass production is reduced without affecting host biomass maintenance. Aggregate results per gene and per gene pair into an essentiality score (count of infected models in which the knockout is essential) and a safety score (count of mock/healthy models in which the knockout is safe). Plot essentiality-vs-safety to pick targets in the essential-but-safe region, e.g., GUK1 among single genes versus more toxic hits like CRLS1 and SGMS1. For downstream drug repositioning, exclude gene pairs where both genes are targeted by the same drug, and discard drugs that are withdrawn, nutraceutical, or experimental.

## Related tools

- **COBRA Toolbox V3** (Runs the in silico single- and double-gene deletion simulations on the context-specific models in MATLAB) — https://github.com/opencobra/cobratoolbox
- **rFASTCORMICS** (Reconstructs the mock and infected context-specific genome-scale metabolic models that the deletion screen is run on) — https://github.com/sysbiolux/rFASTCORMICS
- **MATLAB** (Execution environment for the COBRA Toolbox deletion workflows)

## Examples

```
[gr, eb] = singleDeletion(model); % per model in MATLAB/COBRA V3; repeat with doubleDeletion(model) for gene pairs, then aggregate essentiality/safety counts
```

## Evaluation signals

- Each essential knockout must reduce viral biomass production while leaving human biomass maintenance unaffected in the infected models
- The 50 reconstructed models (28 infected, 22 mock) should be within the reported size ranges: median 3646 metabolites (2465-5088) and 2456.5 reactions (1790-3474)
- Scatter plot must reproduce the Figure 3 qualitative pattern: GUK1 in the essential-but-safe region; CRLS1 and SGMS1 high essentiality but low safety (high toxicity)
- Essentiality score of a gene equals the count of infected models in which its knockout is essential; safety score equals the count of mock models in which it is safe
- Gene pairs where both genes are hit by the same drug must be excluded from the combination list and moved to the single-drug list

## Limitations

- Predicted essentiality requires in vitro confirmation: further single- and double-gene deletion studies are needed to determine the essentiality of the identified single genes and gene pairs
- Targets arise from network effects on the host metabolome (the virus is only modeled through its biomass function) rather than direct docking on the viral proteome
- Findings are limited to the modeled lung cell lines (A549, Calu-3, H1299, NHBE); further tests on other tissues and with other optimization functions, plus in vitro/in vivo validation, are required before considering any predictions as potential drugs

## Evidence

- [readme] rFASTCORMICS README describes the model reconstruction method: [evidence span withheld — anchor status: external_doc]
