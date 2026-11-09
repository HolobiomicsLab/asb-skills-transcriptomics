---
name: sunburst-enrichment-visualization
description: Use when you have InterCellar functional annotation of int-pairs for
  ≥2 conditions and need a per-condition sunburst view of terms significant at p <
  0.05 (e.g., reproducing Fig. 6 for COVID-19 critical vs moderate).
when_to_use_negative:
- The input is not a set of pre-computed CCI results processed through InterCellar's
  function-verse annotation — the sunburst view builds on int-pair functional annotation,
  not raw scRNA-seq counts
- You are working in the cluster-verse or gene-verse (cell cluster or gene filtering/visualization)
  rather than comparing condition-specific functional terms
- You need de novo pathway inference rather than visualization of terms already tested
  by InterCellar's permutation-based enrichment
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: InterCellar
  role: Interactive R/Shiny platform performing the multiple-conditions comparison,
    functional annotation in the function-verse, and sunburst visualization with export
    of plots and tables
  repo: https://github.com/martaint/InterCellar
- name: R
  role: Runtime environment for the Bioconductor InterCellar package (R 4.1, Bioconductor
    3.14)
- name: Shiny
  role: Interactive app framework in which the function-verse and multiple-conditions
    sunburst analysis are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:344b5d3f9856bc5787e0664087ab3b28ee24eff526a9be9aae7cbb9b3ba0b42b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar
  tools:
  - InterCellar
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# sunburst-enrichment-visualization

## Summary

Generate and export a sunburst plot of significantly enriched functional terms per condition within InterCellar's function-verse, using results of a one-sided permutation test (1000 permutations, p < 0.05) on condition-unique int-pairs to visualize condition-specific cell-cell communication.

## When to use

Use when you have InterCellar functional annotation of int-pairs for ≥2 conditions and need a per-condition sunburst view of terms significant at p < 0.05 (e.g., reproducing Fig. 6 for COVID-19 critical vs moderate).

## When NOT to use

- The input is not a set of pre-computed CCI results processed through InterCellar's function-verse annotation — the sunburst view builds on int-pair functional annotation, not raw scRNA-seq counts
- You are working in the cluster-verse or gene-verse (cell cluster or gene filtering/visualization) rather than comparing condition-specific functional terms
- You need de novo pathway inference rather than visualization of terms already tested by InterCellar's permutation-based enrichment

## Inputs

- Pre-computed CCI dataset (inferred ligand-receptor interactions) per condition, from InterCellar-supported tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR)
- Functional annotation of int-pairs in the function-verse (GO and pathway database terms)
- Condition-unique int-pair lists from multiple-conditions comparison
- One-sided permutation test results (1000 permutations) with per-term p-values

## Outputs

- Sunburst plot of enriched functional terms per condition (exported plot file, e.g., Fig. 6)
- Results table of significant terms per condition (p < 0.05), exported for reproducibility

## How to apply

Load the pre-computed CCI datasets for each condition (e.g., COVID-19 critical and moderate, produced by supported tools such as CellChat or CellPhoneDB) into InterCellar and annotate int-pairs with functional terms in the function-verse using Gene Ontology and pathway resources. Compare interactions across conditions to extract int-pairs uniquely found in each condition, then run the one-sided permutation test with 1000 permutations on these condition-unique int-pairs and call terms significant at p < 0.05. In the multiple-conditions analysis, generate the sunburst plot of enriched terms per condition — for the COVID-19 data this confirms 'inflammation mediated by chemokine and cytokine signaling pathway' as significant in critical (cytokine storm signals) and 'T cell costimulation' in moderate. Use InterCellar's download options to export both the plot file and the accompanying results table so the figure can be finalized and reused.

## Related tools

- **InterCellar** (Interactive R/Shiny platform performing the multiple-conditions comparison, functional annotation in the function-verse, and sunburst visualization with export of plots and tables) — https://github.com/martaint/InterCellar
- **R** (Runtime environment for the Bioconductor InterCellar package (R 4.1, Bioconductor 3.14))
- **Shiny** (Interactive app framework in which the function-verse and multiple-conditions sunburst analysis are executed)

## Examples

```
library(InterCellar); InterCellar::run_app( reproducible = TRUE )
```

## Evaluation signals

- The permutation test is run with exactly 1000 permutations and significance is called at p < 0.05
- For the COVID-19 benchmark, the sunburst shows 'inflammation mediated by chemokine and cytokine signaling pathway' significant in critical cases and 'T cell costimulation' significant in moderate cases
- The exported results table matches the terms displayed in the sunburst plot per condition
- The analysis was launched with reproducible = TRUE so results are reproducible across R sessions
- Exported figure and table reproduce Fig. 6 when re-run against the InterCellar-reproducibility repository data

## Limitations

- InterCellar supports only a limited set of input tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); CCI data from other tools must be converted or are unsupported
- Predicted interactions underlying the enrichment require further experimental validation (e.g., smFISH or flow cytometry)
- Relies on pre-computed ligand-receptor interactions; InterCellar does not itself infer CCI from scRNA-seq data

## Evidence

- [abstract] InterCellar implements data-driven analyses including the possibility to compare cell-cell communication from multiple conditions.: "InterCellar implements data-driven analyses including the possibility to compare cell-cell communication from multiple conditions."
- [intro] in the so-called function-verse, InterCellar provides multiple resources to perform functional annotation of enriched interactions: "For example, in the so-called function-verse, InterCellar provides multiple resources to perform functional annotation of enriched interactions."
- [intro] The first enables a focused inspection of cell−cell communication through the analysis of the so-called int-pair modules: "The first enables a focused inspection of cell−cell communication through the analysis of the so-called int-pair modules, which are defined as groups of interactions with similar functional patterns"
- [intro] InterCellar ensures reproducibility of the analysis and facilitates collection and finalization of results by providing multiple download options for tables and figures.: "InterCellar ensures reproducibility of the analysis and facilitates collection and finalization of results by providing multiple download options for tables and figures."
- [readme] The flag reproducible = TRUE ensures that your results will be reproducible across R sessions.: "The flag `reproducible = TRUE` ensures that your results will be reproducible
across R sessions."
