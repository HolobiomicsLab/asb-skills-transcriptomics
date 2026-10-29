---
name: cellphonedb-cci-data-upload
description: Use when you already have pre-computed, inferred ligand-receptor interactions
  from CellPhoneDB (or CellChat, ICELLNET, SingleCellSignalR) and need to load them
  into InterCellar for interactive CCI analysis.
when_to_use_negative:
- Input is raw scRNA-seq count data rather than pre-computed ligand-receptor interactions
  — InterCellar's workflow starts from CCI data, not raw counts
- The CCI output comes from a tool outside InterCellar's supported set (CellChat,
  CellPhoneDB, ICELLNET, SingleCellSignalR) — extension to other tools was only planned
  future work as of the paper
- The analysis requires heavy scripting-based downstream statistics rather than interactive
  exploration — competing approaches or custom R code may be more appropriate
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3308
tools:
- name: InterCellar
  role: 'Target platform: Shiny app into which pre-computed CCI data are uploaded
    and analyzed in the cluster-verse (autocrine/paracrine classification, back-to-back
    barplot)'
  repo: https://github.com/martaint/InterCellar
- name: CellPhoneDB
  role: Source tool that produced the pre-computed ligand-receptor interaction datasets
    uploaded into InterCellar
- name: R
  role: Runtime environment (R 4.1 / Bioconductor 3.14) required to install and launch
    InterCellar
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:bf2df7c037dde21ef0508309e7197aa9a411f5cc546cfd5354e9f8c4c96720b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar
  tools:
  - CellPhoneDB
  - InterCellar
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cellphonedb-cci-data-upload

## Summary

Upload CellPhoneDB-derived cell-cell interaction (CCI) output files into InterCellar as pre-computed ligand-receptor interaction data, so that downstream per-cluster and multi-condition comparisons (e.g., critical vs. moderate COVID-19) can be performed interactively in R/Shiny without programming.

## When to use

Use when you already have pre-computed, inferred ligand-receptor interactions from CellPhoneDB (or CellChat, ICELLNET, SingleCellSignalR) and need to load them into InterCellar for interactive CCI analysis.

## When NOT to use

- Input is raw scRNA-seq count data rather than pre-computed ligand-receptor interactions — InterCellar's workflow starts from CCI data, not raw counts
- The CCI output comes from a tool outside InterCellar's supported set (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR) — extension to other tools was only planned future work as of the paper
- The analysis requires heavy scripting-based downstream statistics rather than interactive exploration — competing approaches or custom R code may be more appropriate

## Inputs

- CellPhoneDB-derived CCI output files (pre-computed inferred ligand-receptor interactions) for each condition, e.g., three COVID-19 datasets (critical vs. moderate)
- Preprocessing R script with manual annotation of selected ligands and receptors (from martaint/InterCellar-reproducibility input_data/preprocessing)

## Outputs

- InterCellar CCI datasets loaded in the app, accessible across cluster-verse, gene-verse, and function-verse
- Per-cluster aggregated interaction counts (autocrine vs. paracrine) per condition dataset
- Back-to-back barplot image (back_to_back_barplot.png) comparing per-cluster interaction counts between conditions

## How to apply

Install InterCellar as a Bioconductor package (R 4.1, Bioconductor 3.14) or via Bioconda/Docker, then launch the Shiny app with InterCellar::run_app(reproducible = TRUE). In the app's first step, upload the pre-computed, inferred ligand-receptor interactions as a CCI dataset — InterCellar builds its entire workflow on these pre-computed inputs rather than raw scRNA-seq data; the reproducibility repo (martaint/InterCellar-reproducibility) provides the COVID-19 input data and preprocessing scripts. Once uploaded, analysis proceeds through three 'universes' (cluster-verse, gene-verse, function-verse); for this skill's downstream comparison, use the cluster-verse to classify each interaction as autocrine (source cluster equals target cluster) or paracrine (source and target differ) and aggregate interaction counts per cell cluster per dataset. Repeat the upload for each of the multiple condition datasets (here, the three COVID-19 CCI datasets) so InterCellar's multi-condition comparison functionality can highlight condition-specific communication patterns.

## Related tools

- **InterCellar** (Target platform: Shiny app into which pre-computed CCI data are uploaded and analyzed in the cluster-verse (autocrine/paracrine classification, back-to-back barplot)) — https://github.com/martaint/InterCellar
- **CellPhoneDB** (Source tool that produced the pre-computed ligand-receptor interaction datasets uploaded into InterCellar)
- **R** (Runtime environment (R 4.1 / Bioconductor 3.14) required to install and launch InterCellar)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("InterCellar"); InterCellar::run_app(reproducible = TRUE)
```

## Evaluation signals

- All three CellPhoneDB-derived CCI datasets upload without format errors and appear in InterCellar's cluster-verse, gene-verse, and function-verse
- Each interaction is classified as autocrine (source cluster equals target cluster) or paracrine (source and target differ), and per-cluster counts sum across all interaction flows
- The back-to-back barplot reproduces the reported pattern: striking gain of interactions for mast cells (MC) and proliferating NKT cells (NKT-p) in critical vs. moderate COVID-19, with higher macrophage and lower T cell counts in critical cases
- The plot is exported reproducibly (reproducible = TRUE) as back_to_back_barplot.png

## Limitations

- InterCellar currently supports only a limited set of input tools (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR); extension to other published tools is planned future work
- InterCellar accepts pre-computed ligand-receptor interactions only — raw scRNA-seq data cannot be uploaded directly, and predicted interactions await experimental validation (e.g., smFISH or flow cytometry)
- Installation may fail on missing dependencies (golem, ComplexHeatmap) and Bioconda/Docker launches may need an explicit browser option (options(browser="firefox"))

## Evidence

- [intro] The workflow of InterCellar builds upon pre-computed, inferred ligand−receptor interactions, which can be uploaded in the first step as a cell−cell interactions dataset (CCI data): "The workflow of InterCellar builds upon pre-computed, inferred ligand−receptor interactions, which can be uploaded in the first step as a cell−cell interactions dataset"
- [intro] either CCI data generated by InterCellar-supported tools (CellChat12, CellPhoneDB11, ICELLNET15, or SingleCellSignalR16): "either CCI data generated by InterCellar-supported tools (CellChat12, CellPhoneDB11, ICELLNET15, or SingleCellSignalR16)"
- [readme] In order to start the app, please run the following command: InterCellar::run_app( reproducible = TRUE ): [evidence span withheld — anchor status: not_found]
- [abstract] InterCellar implements data-driven analyses including the possibility to compare cell-cell communication from multiple conditions.: "InterCellar implements data-driven analyses including the possibility to compare cell-cell communication from multiple conditions."
- [discussion] the analysis suggested a consistent pattern of communication for patients with critical COVID-19, involving cell types such as mast cells: "the analysis suggested a consistent pattern of communication for patients with critical COVID-19, involving cell types such as mast cells"
