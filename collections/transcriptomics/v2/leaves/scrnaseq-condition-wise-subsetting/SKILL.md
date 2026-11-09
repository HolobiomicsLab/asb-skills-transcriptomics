---
name: scrnaseq-condition-wise-subsetting
description: Use when a pooled single-condition-agnostic scRNA-seq matrix with cell-type
  annotations and patient/condition labels must be divided into separate disease-state
  groups before per-group ligand-receptor inference (e.
when_to_use_negative:
- The input is already a set of per-condition CCI datasets or CellPhoneDB output tables
  — subsetting is complete and the next step is InterCellar upload/analysis
- The scRNA-seq data has no condition or patient grouping variable, so no meaningful
  condition-wise split can be made
- The intended CCI tool is not InterCellar-supported (e.g., tools outside CellChat,
  CellPhoneDB, ICELLNET, SingleCellSignalR), since the resulting CCI datasets must
  be loadable into InterCellar
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellPhoneDB
  role: Run separately (v2, default parameters) on each condition subset to infer
    ligand-receptor interactions, producing the per-condition CCI datasets
- name: InterCellar
  role: Downstream consumer of the condition-wise CCI datasets; supports comparison
    of interactions across multiple conditions to highlight condition-specific communication
    patterns
  repo: https://github.com/martaint/InterCellar-reproducibility
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:c10f80f427ccf218b6365ce6e9fb66d44e94805b30f1de9ca3d98759b3df3192
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar-reproducibility
  tools:
  - CellPhoneDB
  - InterCellar
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# scrnaseq-condition-wise-subsetting

## Summary

Splits a pooled scRNA-seq expression matrix plus cell-type annotations into per-condition groups (e.g., control, moderate, critical COVID-19) so that CCI inference such as CellPhoneDB v2 can be run separately on each group, yielding condition-specific CCI input datasets for downstream comparison in InterCellar.

## When to use

Use when a pooled single-condition-agnostic scRNA-seq matrix with cell-type annotations and patient/condition labels must be divided into separate disease-state groups before per-group ligand-receptor inference (e.g., control vs moderate vs critical COVID-19 for InterCellar multi-condition comparison).

## When NOT to use

- The input is already a set of per-condition CCI datasets or CellPhoneDB output tables — subsetting is complete and the next step is InterCellar upload/analysis
- The scRNA-seq data has no condition or patient grouping variable, so no meaningful condition-wise split can be made
- The intended CCI tool is not InterCellar-supported (e.g., tools outside CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR), since the resulting CCI datasets must be loadable into InterCellar

## Inputs

- Preprocessed pooled scRNA-seq expression matrix (e.g., Chua et al. COVID-19 data) with cell-type annotations
- Condition/patient group labels defining control, moderate, and critical subsets
- Reference CCI deposits in martaint/InterCellar-reproducibility for validation

## Outputs

- Three condition-specific CCI datasets (control, moderate, critical), each containing CellPhoneDB output files: mean expression, p-values, and significant means

## How to apply

Load the preprocessed expression matrix and cell-type annotations (here, the Chua et al. COVID-19 scRNA-seq data from the martaint/InterCellar-reproducibility repository). Partition the data by condition: keep the control group intact (5 controls, 2,966 cells) and subsample the larger patient groups to a common size (8 moderate and 11 critical patients each subsampled to 10,000 cells) so group sizes are comparable. Run CellPhoneDB v2 statistical analysis with default parameters independently on each condition group to infer ligand-receptor interactions between cell types. Collect the per-condition outputs (mean expression, p-values, significant means) as the condition-specific CCI datasets. The rationale is that InterCellar compares interactions across multiple datasets to highlight communication patterns unique to each condition, which requires one CCI dataset per condition as upload. Validate by checking each generated CCI dataset against the corresponding reference CCI deposit in the InterCellar-reproducibility repository.

## Related tools

- **CellPhoneDB** (Run separately (v2, default parameters) on each condition subset to infer ligand-receptor interactions, producing the per-condition CCI datasets)
- **InterCellar** (Downstream consumer of the condition-wise CCI datasets; supports comparison of interactions across multiple conditions to highlight condition-specific communication patterns) — https://github.com/martaint/InterCellar-reproducibility

## Evaluation signals

- Exactly three condition-specific CCI datasets exist (control, moderate, critical)
- Control group contains 5 samples with 2,966 cells; moderate (8 patients) and critical (11 patients) groups are each subsampled to 10,000 cells
- Each CCI dataset contains the three CellPhoneDB output components: mean expression, p-values, and significant means
- Each generated CCI dataset matches the corresponding reference CCI deposit in martaint/InterCellar-reproducibility (output_matches_reference)
- CellPhoneDB v2 statistical analysis was run with default parameters — no altered thresholds or non-default options

## Limitations

- Moderate and critical groups were subsampled to 10,000 cells (control was not, at 2,966 cells), so subsampling introduces sampling variability and group sizes are not fully balanced
- CellPhoneDB v2 with default parameters is one of several prediction methods relying on different reference databases and statistical assumptions, so the inferred interactions depend on the tool choice
- InterCellar supports only a limited set of input tools; CCI data from unsupported tools cannot be used downstream without extension

## Evidence

- [abstract] multi-condition comparison: "InterCellar implements data-driven analyses including the possibility to compare cell-cell communication from multiple conditions."
- [intro] condition-specific patterns: "functionalities to compare interactions across multiple datasets, highlighting patterns of communication that are uniquely found in each of the conditions considered"
- [discussion] critical COVID-19 pattern: "the analysis suggested a consistent pattern of communication for patients with critical COVID-19, involving cell types such as mast cells"
- [readme] reproducibility repo input data: [evidence span withheld — anchor status: external_doc]
