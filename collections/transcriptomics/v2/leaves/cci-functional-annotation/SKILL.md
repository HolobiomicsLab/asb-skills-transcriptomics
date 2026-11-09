---
name: cci-functional-annotation
description: Use when you have pre-computed ligand-receptor interactions from one
  or more conditions (e.g., CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR output)
  and need functional annotation of int-pairs plus statistically significant condition-specific
  terms.
when_to_use_negative:
- Input is raw scRNA-seq count data with no pre-computed ligand-receptor interactions
  — InterCellar builds on pre-computed inferred interactions, so an inference tool
  must run first
- Input CCI data comes from tools outside InterCellar's supported set (extension to
  other published tools is planned future work)
- Goal is de novo pathway analysis on a gene list without cell-cell interaction context
  — the function-verse annotates int-pairs, not arbitrary gene sets
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: InterCellar
  role: R/Shiny app providing the function-verse functional annotation of int-pairs,
    multiple-conditions comparison, one-sided permutation test, and sunburst visualization
  repo: https://github.com/martaint/InterCellar
- name: R/Shiny
  role: Implementation language/framework in which InterCellar's interactive function-verse
    analysis is delivered
- name: CellChat
  role: Supported upstream tool whose inferred interactions can be uploaded as CCI
    input for annotation
  repo: sqjin/CellChat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:06e99bf4525d6437cc5191000eef14ffffc34a23e9604f2514c8d4f44839aadb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar
  tools:
  - CellChat
  - InterCellar
  - R
  - R/Shiny
  - Shiny
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# cci-functional-annotation

## Summary

InterCellar's function-verse workflow annotates predicted ligand-receptor int-pairs with Gene Ontology/pathway functional terms, and extends this to a one-sided permutation test (1000 permutations, p < 0.05) on condition-unique int-pairs to identify condition-specific functional terms across multiple CCI datasets.

## When to use

Use when you have pre-computed ligand-receptor interactions from one or more conditions (e.g., CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR output) and need functional annotation of int-pairs plus statistically significant condition-specific terms.

## When NOT to use

- Input is raw scRNA-seq count data with no pre-computed ligand-receptor interactions — InterCellar builds on pre-computed inferred interactions, so an inference tool must run first
- Input CCI data comes from tools outside InterCellar's supported set (extension to other published tools is planned future work)
- Goal is de novo pathway analysis on a gene list without cell-cell interaction context — the function-verse annotates int-pairs, not arbitrary gene sets

## Inputs

- Pre-computed CCI dataset (inferred ligand-receptor interactions) from InterCellar-supported tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR)
- Two or more condition-specific CCI datasets for multiple-conditions comparison (e.g., critical vs moderate COVID-19)

## Outputs

- Functional annotation terms assigned to int-pairs (from Gene Ontology and pathway databases)
- List of condition-unique int-pairs per condition
- Permutation test results table of significant functional terms (p < 0.05, 1000 permutations)
- Sunburst plot of enriched terms per condition (exportable plot file and results table)

## How to apply

Upload the pre-computed inferred ligand-receptor interactions as CCI datasets into InterCellar (launched via InterCellar::run_app with reproducible = TRUE for session-stable results). In the function-verse, run functional annotation of int-pairs using Gene Ontology and pathway databases to assign biological function terms to each interaction. For multiple-conditions analysis, compare interactions across datasets to extract int-pairs uniquely found in each condition (e.g., the CD160 & TNFRSF14 pair in moderate COVID-19). Apply a one-sided permutation test with 1000 permutations on the condition-unique int-pairs and call terms significant at p < 0.05. The rationale is that condition-unique int-pairs, not shared ones, carry the condition-specific communication signal, and permutation testing against them controls for functional term background enrichment.

## Related tools

- **InterCellar** (R/Shiny app providing the function-verse functional annotation of int-pairs, multiple-conditions comparison, one-sided permutation test, and sunburst visualization) — https://github.com/martaint/InterCellar
- **R/Shiny** (Implementation language/framework in which InterCellar's interactive function-verse analysis is delivered)
- **CellChat** (Supported upstream tool whose inferred interactions can be uploaded as CCI input for annotation) — sqjin/CellChat

## Examples

```
Rscript -e "library(InterCellar); InterCellar::run_app(reproducible=TRUE)"
```

## Evaluation signals

- Functional term called significant only at p < 0.05 under the one-sided permutation test with 1000 permutations on condition-unique int-pairs
- For the COVID-19 critical vs moderate comparison, 'inflammation mediated by chemokine and cytokine signaling pathway' must be significant in critical and 'T cell costimulation' in moderate
- Condition-unique int-pair extraction must exclude shared interactions (e.g., CD160 & TNFRSF14 flagged as unique to moderate)
- Reproducibility: run_app(reproducible = TRUE) so results are stable across R sessions
- Deliverables include both the exported sunburst plot file and the accompanying results table

## Limitations

- InterCellar currently supports only a limited set of input tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); extension to other published tools is planned future work
- Predicted interactions await experimental validation (e.g., smFISH or flow cytometry co-occurrence), so annotated significant terms rest on predicted int-pairs
- There is a lack of a standardized approach for downstream analysis/interpretation of cell-cell communication results — permutation-test parameters (1000 permutations, one-sided) are as implemented in this workflow

## Evidence

- [intro] For example, in the so-called function-verse, InterCellar provides multiple resources to perform functional annotation of enriched interactions.: "For example, in the so-called function-verse, InterCellar provides multiple resources to perform functional annotation of enriched interactions."
- [abstract] InterCellar implements data-driven analyses including the possibility to compare cell-cell communication from multiple conditions.: "InterCellar implements data-driven analyses including the possibility to compare cell-cell communication from multiple conditions."
- [discussion] an in-depth analysis of cellular interactions in COVID-19 patients confirmed the primary role of immune response in discriminating between different courses of the disease: "an in-depth analysis of cellular interactions in COVID-19 patients confirmed the primary role of immune response in discriminating between different courses of the disease"
- [readme] Moreover, based on functional annotation from Gene Ontology and pathway databases, `InterCellar` implements data-driven analyses to investigate cell-cell communication in one or multiple conditions.: [evidence span withheld — anchor status: external_doc]
