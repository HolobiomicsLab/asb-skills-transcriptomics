---
name: factor-disease-association-statistics
description: Use when a trained MOFA/scPAFA model over pseudobulk pathway activity
  scores must be linked to disease state — i.e., you have sample-level clinical metadata
  (SLE vs healthy) and latent factors whose variance needs statistical testing against
  condition.
when_to_use_negative:
- The study has fewer than 15 samples (not cells), the minimum required to use the
  MOFA model for identifying multicellular pathway modules.
- The goal is to infer biological causal effects rather than data-driven associations
  of pathways and cell types within a module.
- Interpretability across many disease states is required, since interpretability
  may decrease with multiple disease states.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scPAFA
  role: Python library whose step-4 downstream analysis tests MOFA factors against
    clinical metadata to identify disease-related multicellular pathway modules
  repo: https://github.com/ZhuoliHuang/scPAFA
- name: mofapy2
  role: Trains the multi-group MOFA+ model whose latent factors are statistically
    tested against disease status
  repo: https://github.com/bioFAM/mofapy2
- name: MOFA
  role: Framework capturing variance in PAS among samples into interpretable latent
    factors
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:4f05f159985afdab99640551cccf4393834da55284743ce079d7cf695b224d3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuoliHuang/scPAFA
  tools:
  - MOFA
  - Python
  - UCell
  - mofapy2
  - scPAFA
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# factor-disease-association-statistics

## Summary

Statistically tests MOFA latent factors from a trained scPAFA model against sample-level clinical metadata to identify disease-related multicellular pathway modules (e.g., factor 1 elevated and factor 6 decreased in SLE vs healthy controls in the lupus PBMC atlas).

## When to use

Use when a trained MOFA/scPAFA model over pseudobulk pathway activity scores must be linked to disease state — i.e., you have sample-level clinical metadata (SLE vs healthy) and latent factors whose variance needs statistical testing against condition.

## When NOT to use

- The study has fewer than 15 samples (not cells), the minimum required to use the MOFA model for identifying multicellular pathway modules.
- The goal is to infer biological causal effects rather than data-driven associations of pathways and cell types within a module.
- Interpretability across many disease states is required, since interpretability may decrease with multiple disease states.

## Inputs

- Trained MOFA/mofapy2 model with latent factors over pseudobulk PAS samples
- Sample-level clinical metadata (e.g., SLE vs healthy control status, donor/disease labels)
- Factor weight matrix of pathway-cell type features

## Outputs

- Disease-related multicellular pathway modules (significant latent factors with corresponding pathway weights across cell types)
- Statistical test results for factor-disease association (e.g., factor 1 elevated, factor 6 decreased in SLE)
- Visualizations of high-weighted pathways and sample/donor stratification (Figures 5 and 6a-e)

## How to apply

After training a multi-group MOFA+ model on the pseudobulk pathway activity score (PAS) matrix (e.g., 175 training samples aggregated from 941,542 high-quality lupus PBMC cells scored for 1383 NCATS BioPlanet pathways via 'fast_Ucell'), join the factor values to sample-level clinical metadata. Run downstream statistical analysis testing whether each latent factor differs significantly between disease and control groups — for the lupus atlas, factor 1 should be significantly increased and factor 6 significantly decreased in SLE patients. Then extract and visualize the high-weighted pathway–cell type pairs of significant factors, confirming known biology (interferon alpha/beta, Type II interferon, Granzyme A among top-weighted features, reproducing Figures 5 and 6a–e). Modules are data-driven associations, not causal effects, and interpretation should account for MOFA being a linear model.

## Related tools

- **scPAFA** (Python library whose step-4 downstream analysis tests MOFA factors against clinical metadata to identify disease-related multicellular pathway modules) — https://github.com/ZhuoliHuang/scPAFA
- **mofapy2** (Trains the multi-group MOFA+ model whose latent factors are statistically tested against disease status) — https://github.com/bioFAM/mofapy2
- **MOFA** (Framework capturing variance in PAS among samples into interpretable latent factors)

## Examples

```
from mofapy2.run.entry_point import entry_point; ent = entry_point(); ent.run(); factors = model.get_factors(); test_factor_disease_association(factors, clinical_metadata, disease_col='SLE')
```

## Evaluation signals

- In the lupus atlas validation, factor 1 must be significantly increased and factor 6 significantly decreased in SLE patients versus healthy controls.
- Interferon alpha/beta, Type II interferon, and Granzyme A pathway-cell type pairs must appear among the top-weighted features of significant factors, matching Figures 5 and 6a-e.
- The PAS matrix must have been aggregated to a sufficient number of training samples (e.g., 175 pseudobulk samples) before factor testing.
- Significant factors should show biologically interpretable high-weighted pathways consistent with previous reports.

## Limitations

- scPAFA requires studies to include at least 15 samples (not cells) for MOFA-based multicellular pathway module identification.
- Pseudobulk integration of the PAS matrix leads to information loss since PAS of multiple cells is aggregated.
- MOFA is a linear model; interpretability is achieved by sacrificing information content in each factor, and interpretability may decrease with multiple disease states.
- Pathways and cell types within the same module suggest data-driven associations only, not biological causal effects.

## Evidence

- [other] downstream statistical analysis: [evidence span withheld — anchor status: not_found]
- [readme] MOFA model training rationale: "MOFA model is trained to capture variance in PAS among different samples."
- [discussion] minimum sample requirement: "scPAFA requires studies to include at least 15 samples (not cells), which is the requirement of using the MOFA model for identifying multicellular pathway modules."
- [discussion] data-driven associations only: "the pathways and cell types within the same module can only suggest data-driven associations, rather than determining biological causal effects."
