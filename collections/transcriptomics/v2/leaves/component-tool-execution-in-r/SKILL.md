---
name: component-tool-execution-in-r
description: Use when you need independent cell-type label predictions from multiple
  component classifiers on a scRNA-seq gene expression matrix, to feed majority-vote
  consensus or GCN label propagation (e.g. scSHARP's 0.51 threshold).
when_to_use_negative:
- Input is already a consolidated multi-tool prediction file — the consensus/label-propagation
  step, not per-tool execution, is what is needed.
- Only a single classifier's output is required, or the expression matrix contains
  processed/normalized values rather than raw counts.
- Working purely in Python without any need for R component-tool predictions — the
  Python-only scSHARP GCN tool still requires the R tools as a dependency ('You will
  need one R dependency to run scSHARP').
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: R4scSHARP
  role: R package whose main function run_r4scsharp executes the five component classification
    tools and consolidates their predictions
  repo: https://github.com/W-Holtz/R4scSHARP
- name: SingleR
  role: Component classifier executed within R4scSHARP; installed via BiocManager
  repo: https://github.com/W-Holtz/R4scSHARP
- name: scSorter
  role: Component classifier executed within R4scSHARP; installed from CRAN
  repo: https://github.com/W-Holtz/R4scSHARP
- name: scPred
  role: Component classifier executed within R4scSHARP; installed from GitHub after
    harmony
  repo: https://github.com/W-Holtz/R4scSHARP
- name: SCINA
  role: Component classifier executed within R4scSHARP; installed from CRAN
  repo: https://github.com/W-Holtz/R4scSHARP
- name: R
  role: Runtime environment for R4scSHARP; installed with conda if R4scSHARP is not
    driven from Python
- name: conda
  role: Environment manager used to create a Python 3.9 environment and install R
    alongside dependencies when running scSHARP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad360
    title: Consensus label propagation with graph convolutional networks for single-cell
      RNA sequencing cell type annotation
  source_content_hash: sha256:6ebbff2f7526717834c071528f2f40d77794e496aabdf053545c22b74a069b44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/W-Holtz/R4scSHARP
  tools:
  - Python
  - R
  - R4scSHARP
  - SCINA
  - SingleR
  - conda
  - scPred
  - scSHARP
  - scSorter
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# component-tool-execution-in-r

## Summary

Runs a suite of five single-cell classification tools (SCINA, ScType, scSorter, SingleR, scPred) on an RNA gene expression matrix via the R4scSHARP R package, returning per-tool cell-type label predictions in a single consolidated file for downstream consensus aggregation.

## When to use

Use when you need independent cell-type label predictions from multiple component classifiers on a scRNA-seq gene expression matrix, to feed majority-vote consensus or GCN label propagation (e.g. scSHARP's 0.51 threshold).

## When NOT to use

- Input is already a consolidated multi-tool prediction file — the consensus/label-propagation step, not per-tool execution, is what is needed.
- Only a single classifier's output is required, or the expression matrix contains processed/normalized values rather than raw counts.
- Working purely in Python without any need for R component-tool predictions — the Python-only scSHARP GCN tool still requires the R tools as a dependency ('You will need one R dependency to run scSHARP').

## Inputs

- Single-cell RNA-seq gene expression matrix with raw counts (e.g. the Splatter-simulated, mouse testis Jung et al. 2019, and human PBMC Zheng et al. 2017 benchmark matrices)
- R environment with R4scSHARP and its four suggested dependencies (SingleR, scSorter, scPred, SCINA) installed

## Outputs

- A single file of per-tool cell-type predictions from up to five classification tools, in a succinct format produced by run_r4scsharp
- Downstream: per-dataset counts/fractions of unconfidently labeled cells written to a CSV summary table (e.g. 15.6% / 8.5% / 6.6% / 3.0% for the four benchmarks)

## How to apply

Install the R4scSHARP package from GitHub with devtools::install_github('W-Holtz/R4scSHARP'), then install the four suggested dependencies separately (SingleR via BiocManager, scSorter and SCINA via CRAN install.packages, scPred via devtools from powellgenomicslab/scPred after harmony). Load raw counts (not normalized/processed data — the scSHARP_tool README says 'Please input raw counts') and call the package's main function run_r4scsharp, which runs up to the five cell classification tools on the RNA gene expression matrix and returns the tool predictions in a succinct combined file. The per-tool predictions are then aggregated per cell so a consensus step can mark cells unconfident when fewer than a 0.51 fraction of the five tools agree on a label. Decision point: run the tools via R directly, or via Python against the same predictions by pip-installing scSHARP into a Python 3.9 conda environment.

## Related tools

- **R4scSHARP** (R package whose main function run_r4scsharp executes the five component classification tools and consolidates their predictions) — https://github.com/W-Holtz/R4scSHARP
- **SingleR** (Component classifier executed within R4scSHARP; installed via BiocManager) — https://github.com/W-Holtz/R4scSHARP
- **scSorter** (Component classifier executed within R4scSHARP; installed from CRAN) — https://github.com/W-Holtz/R4scSHARP
- **scPred** (Component classifier executed within R4scSHARP; installed from GitHub after harmony) — https://github.com/W-Holtz/R4scSHARP
- **SCINA** (Component classifier executed within R4scSHARP; installed from CRAN) — https://github.com/W-Holtz/R4scSHARP
- **R** (Runtime environment for R4scSHARP; installed with conda if R4scSHARP is not driven from Python)
- **conda** (Environment manager used to create a Python 3.9 environment and install R alongside dependencies when running scSHARP)

## Examples

```
library(devtools); install_github("W-Holtz/R4scSHARP"); library(R4scSHARP)
```

## Evaluation signals

- The consolidated prediction file contains a label from each of the up-to-five tools (SCINA, ScType, scSorter, SingleR, scPred) for every cell in the input matrix.
- After applying the 0.51 majority-vote threshold, unconfident fractions on the four benchmarks should be comparable to the reported values: 15.6% (de.facScale 0.7 simulation), 8.5% (de.facScale 0.8), 6.6% / 46 cells (testis), 3.0% / 2310 cells (PBMC).
- Package installation succeeds only when all four suggested dependencies (SingleR, scSorter, scPred, SCINA) are present; failure of any BiocManager/devtools/CRAN install is a sign the setup is incomplete.
- Input check: the matrix passed to run_r4scsharp contains raw counts, per the scSHARP_tool README instruction 'Please input raw counts.'

## Limitations

- The four suggested dependencies (SingleR, scSorter, scPred, SCINA) are not installed by R4scSHARP itself and must be installed individually.
- R4scSHARP is only available on GitHub (not CRAN at the time of the README), and the README notes 'More documentation is coming!' with only an examples R script showcasing basic functionality.
- Additional PyTorch Geometric dependencies such as torch-cluster may need manual installation if the tools are driven from Python via pip-installed scSHARP.
- Version numbers for R and the component tools are not specified in the article's methods.

## Evidence

- [readme] the main function, run_r4scsharp, is used to run up to five cell classification tools on a RNA gene expression matrix, then return the tool predictions in a succinct file: [evidence span withheld — anchor status: external_doc]
- [methods] Our package utilizes a suite of cutting edge single cell classification tools which we have compiled as an R package: "Our package utilizes a suite of cutting edge single cell classification tools which we have compiled as an R package"
- [readme] Our package contains 4 suggested dependencies that need to be installed on their own.: [evidence span withheld — anchor status: external_doc]
- [readme] You will need one R dependency to run scSHARP. Repository and install instructions can be found here: [evidence span withheld — anchor status: external_doc]
- [readme] Please input raw counts.: "See demo.ipynb for example work flow.
Please input raw counts."
