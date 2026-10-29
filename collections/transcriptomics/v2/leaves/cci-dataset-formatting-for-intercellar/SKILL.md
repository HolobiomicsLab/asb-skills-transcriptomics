---
name: cci-dataset-formatting-for-intercellar
description: Use when InterCellar analysis is required but no CCI dataset exists yet,
  and ligand-receptor interactions must be pre-computed with CellPhoneDB v2 from a
  scRNA-seq expression matrix with cell-type annotations, per condition.
when_to_use_negative:
- A CCI dataset already exists in an InterCellar-supported format (CellChat, CellPhoneDB,
  ICELLNET, SingleCellSignalR) — it can be uploaded directly without re-running inference
- The scRNA-seq data are raw/unprocessed — preprocessing and cell-type annotation
  must be performed first in a separate step
- The target tool is not among InterCellar's supported input tools — the paper notes
  the tool set is limited, with extension to other published tools left as planned
  future work
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: CellPhoneDB
  role: Run in v2 with default statistical-analysis parameters, separately on each
    condition group, to infer the ligand-receptor interactions that become the CCI
    input datasets
- name: InterCellar
  role: Downstream consumer of the generated CCI datasets; the datasets must match
    the format InterCellar accepts in its first workflow step
  repo: https://github.com/martaint/InterCellar
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-021-02986-2
    title: InterCellar enables interactive analysis and exploration of cell−cell communication
      in single-cell transcriptomic data
  source_content_hash: sha256:a2890abad685c4f8766b4d73c1e291d0fcd7693b8114634c0c56e78017e6fe2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/martaint/InterCellar
  tools:
  - CellPhoneDB
  - InterCellar
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cci-dataset-formatting-for-intercellar

## Summary

Reproduce condition-specific cell-cell interaction (CCI) input datasets for InterCellar by running CellPhoneDB v2 on the Chua et al. COVID-19 scRNA-seq data split into control, moderate, and critical patient groups, yielding three separate CCI datasets validated against the InterCellar-reproducibility reference deposit.

## When to use

Use when InterCellar analysis is required but no CCI dataset exists yet, and ligand-receptor interactions must be pre-computed with CellPhoneDB v2 from a scRNA-seq expression matrix with cell-type annotations, per condition.

## When NOT to use

- A CCI dataset already exists in an InterCellar-supported format (CellChat, CellPhoneDB, ICELLNET, SingleCellSignalR) — it can be uploaded directly without re-running inference
- The scRNA-seq data are raw/unprocessed — preprocessing and cell-type annotation must be performed first in a separate step
- The target tool is not among InterCellar's supported input tools — the paper notes the tool set is limited, with extension to other published tools left as planned future work

## Inputs

- Preprocessed Chua et al. COVID-19 scRNA-seq expression matrix (from martaint/InterCellar-reproducibility)
- Cell-type annotations for the Chua et al. dataset
- Condition grouping of patients: 5 controls, 8 moderate, 11 critical

## Outputs

- Three condition-specific CCI datasets (control, moderate, critical) in CellPhoneDB output format
- Per-condition CellPhoneDB output files: mean expression values, p-values, and significant means

## How to apply

Load the preprocessed Chua et al. scRNA-seq expression matrix and cell-type annotations from the martaint/InterCellar-reproducibility repository. Split the data into three condition groups: 5 controls (2,966 cells), 8 moderate and 11 critical patients (each subsampled to 10,000 cells). Run CellPhoneDB v2 statistical analysis with default parameters separately on each condition group to infer ligand-receptor interactions between cell types. Collect the per-condition CellPhoneDB outputs (means, p-values, significant means) as the three CCI datasets, since InterCellar's workflow builds upon pre-computed, inferred ligand-receptor interactions uploaded in the first step as a CCI dataset. Validate each generated dataset against the corresponding reference CCI deposit in the InterCellar-reproducibility repository.

## Related tools

- **CellPhoneDB** (Run in v2 with default statistical-analysis parameters, separately on each condition group, to infer the ligand-receptor interactions that become the CCI input datasets)
- **InterCellar** (Downstream consumer of the generated CCI datasets; the datasets must match the format InterCellar accepts in its first workflow step) — https://github.com/martaint/InterCellar

## Examples

```
cellphonedb method statistical_analysis meta_control.txt counts_control.h5ad --output-path cci_control/ --counts-data hgnc_symbol
```

## Evaluation signals

- Each of the three condition-specific CCI datasets (control, moderate, critical) exists as a separate output
- Each generated CCI dataset matches the corresponding reference CCI deposit in the martaint/InterCellar-reproducibility repository (output_matches_reference)
- Condition splits have the expected composition: 5 controls with 2,966 cells; moderate and critical groups each subsampled to 10,000 cells
- CellPhoneDB was run with default statistical-analysis parameters, so output files (means, p-values, significant means) are present per condition
- The CCI files load successfully in InterCellar as its first-step CCI data upload

## Limitations

- InterCellar currently supports only a limited set of input tools; outputs from other published CCI methods may not be accepted
- Predicted ligand-receptor interactions are computational predictions and still await experimental validation (e.g., smFISH or flow cytometry co-occurrence)
- The moderate and critical groups are subsampled to 10,000 cells, so results depend on the subsampling; per-condition datasets were generated by running CellPhoneDB separately per group

## Evidence

- [intro] Workflow step 1 of InterCellar requires pre-computed interactions: "The workflow of InterCellar builds upon pre-computed, inferred ligand−receptor interactions, which can be uploaded in the first step as a cell−cell interactions dataset (CCI data)"
- [other] Source-card finding on CellPhoneDB v2 generation of three CCI datasets: [evidence span withheld — anchor status: not_found]
- [readme] Reproducibility repository hosts the input data and processing code: [evidence span withheld — anchor status: not_found]
- [discussion] Limitation on supported input tools: "we plan to further extend the input options to other published tools"
