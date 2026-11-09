---
name: permutation-based-enrichment-testing
description: Use when you have pre-computed CCI datasets from two or more conditions
  (e.g., critical and moderate COVID-19) and need to call functional terms significant
  based on condition-unique ligand-receptor int-pairs.
when_to_use_negative:
- You only have a single condition/dataset — the test requires condition-unique int-pairs
  from a multiple-conditions comparison
- You need the CCI interactions themselves inferred from scratch (InterCellar builds
  on pre-computed ligand-receptor interactions, not raw scRNA-seq counts)
- A standardized parametric enrichment test on a gene list is sufficient and no per-condition
  interaction specificity is required
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: InterCellar
  role: Interactive R/Shiny platform whose function-verse performs functional annotation
    of int-pairs and the one-sided permutation test on condition-unique int-pairs
  repo: https://github.com/martaint/InterCellar
- name: R/Shiny
  role: Implementation environment for InterCellar; the analysis is executed interactively
    in the Shiny app without programming skills
  repo: https://github.com/martaint/InterCellar
- name: InterCellar-reproducibility
  role: Supplementary repository with input data and intercellar_walkthrough code
    to reproduce the COVID-19 multiple-conditions results
  repo: https://github.com/martaint/InterCellar-reproducibility
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:4927453eba4efbd8ae0690d3978afc9d758fd1ca8a1984ce735841b5357ccaf7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar
  tools:
  - InterCellar
  - InterCellar-reproducibility
  - R
  - R/Shiny
  - Shiny
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# permutation-based-enrichment-testing

## Summary

Applies a one-sided permutation test (1000 permutations, significance at p < 0.05) to int-pairs uniquely found in each condition, as implemented in InterCellar's function-verse, to identify significant functional terms distinguishing conditions (e.g., critical vs. moderate COVID-19).

## When to use

Use when you have pre-computed CCI datasets from two or more conditions (e.g., critical and moderate COVID-19) and need to call functional terms significant based on condition-unique ligand-receptor int-pairs.

## When NOT to use

- You only have a single condition/dataset — the test requires condition-unique int-pairs from a multiple-conditions comparison
- You need the CCI interactions themselves inferred from scratch (InterCellar builds on pre-computed ligand-receptor interactions, not raw scRNA-seq counts)
- A standardized parametric enrichment test on a gene list is sufficient and no per-condition interaction specificity is required

## Inputs

- Pre-computed CCI datasets (inferred ligand-receptor interactions) for each condition (e.g., critical and moderate COVID-19)
- Functional annotation of int-pairs from the function-verse (GO/pathway terms)
- Condition-unique int-pair sets extracted by cross-condition comparison

## Outputs

- Table of significant functional terms per condition (p < 0.05, 1000 permutations)
- Sunburst plot of enriched terms per condition
- Exported plot files and results table for reproducibility

## How to apply

Load the pre-computed CCI datasets for each condition into InterCellar and, in the function-verse, perform functional annotation of int-pairs to assign Gene Ontology/pathway-based biological function terms. Compare interactions across conditions to extract int-pairs uniquely found in each condition (e.g., the CD160 & TNFRSF14 pair in moderate COVID-19). Run the one-sided permutation test with 1000 permutations on these condition-unique int-pairs and call terms significant at p < 0.05. Verify the expected directionality: in the InterCellar COVID-19 analysis, 'inflammation mediated by chemokine and cytokine signaling pathway' was significant in critical cases (supporting a cytokine storm signal) while 'T cell costimulation' was significant in moderate cases. Visualize results with a sunburst plot of enriched terms per condition and export both the plot and the results table for reproducibility.

## Related tools

- **InterCellar** (Interactive R/Shiny platform whose function-verse performs functional annotation of int-pairs and the one-sided permutation test on condition-unique int-pairs) — https://github.com/martaint/InterCellar
- **R/Shiny** (Implementation environment for InterCellar; the analysis is executed interactively in the Shiny app without programming skills) — https://github.com/martaint/InterCellar
- **InterCellar-reproducibility** (Supplementary repository with input data and intercellar_walkthrough code to reproduce the COVID-19 multiple-conditions results) — https://github.com/martaint/InterCellar-reproducibility

## Examples

```
BiocManager::install("InterCellar"); InterCellar::run_app(reproducible = TRUE)
```

## Evaluation signals

- Permutation test used exactly 1000 permutations and significance threshold p < 0.05, one-sided, on condition-unique int-pairs only
- 'inflammation mediated by chemokine and cytokine signaling pathway' is called significant in critical COVID-19 cases
- 'T cell costimulation' is called significant in moderate COVID-19 cases
- The CD160 & TNFRSF14 pair appears in the moderate-condition-unique int-pair set
- Sunburst plot and results table are exported per condition and match the manuscript figure (Fig. 6)

## Limitations

- InterCellar supports only a limited set of input CCI tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); extension to other tools is planned future work
- Predicted interactions await experimental validation (e.g., smFISH or flow cytometry co-occurrence), so significant terms rest on unvalidated predictions
- Prediction methods rely on different reference databases and statistical assumptions, making upstream tool choice critical for the int-pair inputs

## Evidence

- [intro] functionalities to compare interactions across multiple datasets, highlighting patterns of communication that are uniquely found in each of the conditions considered: "functionalities to compare interactions across multiple datasets, highlighting patterns of communication that are uniquely found in each of the conditions considered"
- [intro] InterCellar ensures reproducibility of the analysis and facilitates collection and finalization of results by providing multiple download options for tables and figures.: "InterCellar ensures reproducibility of the analysis and facilitates collection and finalization of results by providing multiple download options for tables and figures."
- [intro] For example, in the so-called function-verse, InterCellar provides multiple resources to perform functional annotation of enriched interactions.: "For example, in the so-called function-verse, InterCellar provides multiple resources to perform functional annotation of enriched interactions."
- [intro] InterCellar builds upon pre-computed, inferred ligand−receptor interactions, which can be uploaded in the first step as a cell−cell interactions dataset (CCI data): "The workflow of InterCellar builds upon pre-computed, inferred ligand−receptor interactions, which can be uploaded in the first step as a cell−cell interactions dataset (CCI data)"
- [discussion] an in-depth analysis of cellular interactions in COVID-19 patients confirmed the primary role of immune response in discriminating between different courses of the disease: "an in-depth analysis of cellular interactions in COVID-19 patients confirmed the primary role of immune response in discriminating between different courses of the disease6"
- [readme] README: InterCellar runs on your local machine with reproducible results via run_app(reproducible = TRUE); the reproducibility repo contains input data and walkthrough for COVID-19 analyses.: "The flag `reproducible = TRUE` ensures that your results will be reproducible
across R sessions."
