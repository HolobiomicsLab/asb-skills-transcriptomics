---
name: r-package-installation-from-github
description: Use when a required R package (e.g. celexalvrR for converting Seurat/AnnData
  objects to CellexalVR input files) is not on CRAN and must be installed from a GitHub
  repository under R >= 4.0.0.
when_to_use_negative:
- The package is already available on CRAN and can be installed with a plain install.packages()
  call
- The R environment is older than R 4.0.0, under which cellexalvrR is not productively
  used
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: cellexalvrR
  role: Target R package installed from GitHub; preps single-cell expression data
    for CellexalVR and provides back-end in-session calculation functions
  repo: https://github.com/sonejilab/cellexalvrR
- name: R
  role: Runtime environment (>= 4.0.0) in which the GitHub package is installed and
    loaded; CellexalVR calls it via Rscript.exe
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:fc7b2c687fb0f2ead65c490775020d3d019935fafabe0a87cbb196a7c22e92e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/cellexalvrR
  tools:
  - R
  - SCANPY
  - cellexalvrR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# r-package-installation-from-github

## Summary

Install an R package directly from its GitHub repository using devtools::install_github, as required for the cellexalvrR package that prepares single-cell data for CellexalVR and provides in-session back-end calculations.

## When to use

Use when a required R package (e.g. celexalvrR for converting Seurat/AnnData objects to CellexalVR input files) is not on CRAN and must be installed from a GitHub repository under R >= 4.0.0.

## When NOT to use

- The package is already available on CRAN and can be installed with a plain install.packages() call
- The R environment is older than R 4.0.0, under which cellexalvrR is not productively used

## Inputs

- GitHub repository identifier for the R package (e.g. sonejilab/cellexalvrR)
- Local R installation of version 4.0.0 or later
- R 'devtools' package

## Outputs

- Installed and loadable cellexalvrR R package with as_cellexalvrR and export2cellexalvr functions available
- Optionally, pandoc-backed HTML report generation capability for CellexalVR sessions

## How to apply

First ensure R 4.0.0 or later is installed (cellexalvrR is productively used under R 4.0.2 and 4.1.0) and that the devtools package is available, installing it with install.packages("devtools") if needed. Then run devtools::install_github with the GitHub 'owner/repo' identifier, e.g. devtools::install_github("sonejilab/cellexalvrR"), and load the package with library(cellexalvrR). If HTML report generation from CellexalVR sessions is desired, additionally install pandoc, which the reports depend on. Because CellexalVR invokes R through Rscript.exe, note the path to your R installation (e.g. the RscriptFilePath entry in Config/config.xml) so the installed package is reachable from the VR application. Verify the installation by loading the library and checking that the key conversion functions (as_cellexalvrR, export2cellexalvr) are available before attempting data export.

## Related tools

- **cellexalvrR** (Target R package installed from GitHub; preps single-cell expression data for CellexalVR and provides back-end in-session calculation functions) — https://github.com/sonejilab/cellexalvrR
- **R** (Runtime environment (>= 4.0.0) in which the GitHub package is installed and loaded; CellexalVR calls it via Rscript.exe)

## Examples

```
install.packages("devtools"); devtools::install_github("sonejilab/cellexalvrR"); library(cellexalvrR)
```

## Evaluation signals

- library(cellexalvrR) loads without error in an R >= 4.0.0 session
- as_cellexalvrR and export2cellexalvr functions are present after installation
- devtools::install_github completes without dependency-resolution failures under devtools installed via install.packages("devtools")
- CellexalVR can locate the R installation (RscriptFilePath resolvable, e.g. pointing to Rscript.exe under the R install directory)
- If HTML reports are needed, pandoc is installed and callable from the same environment

## Limitations

- cellexalvrR is documented as productively used under R 4.0.2 and 4.1.0; other R versions may not be supported
- Installation from GitHub requires the devtools package and an internet connection to the repository (github.com/sonejilab/cellexalvrR)
- HTML report generation additionally requires a separate pandoc installation

## Evidence

- [readme] Install the CellexalVR R package by running devtools::install_github("sonejilab/cellexalvrr") in an R terminal.: "Install the [CellexalVR R package](https://www.github.com/sonejilab/cellexalvrr) by running `devtools::install_github("sonejilab/cellexalvrr")` in an R terminal."
- [readme] cellexalvrR is productively used under R 4.0.2 and 4.1.0.: [evidence span withheld — anchor status: external_doc]
- [readme] cellexalvrR is an R package to prep single-cell expression data for exploration/use in our virtual realitiy data analysis platform CellexalVR, and also to provide back-end functions that perform in-session calculations.: [evidence span withheld — anchor status: external_doc]
- [readme] If you want to make use of the html reports, you need to install pandoc.: "If you want to make use of the html reports, you need to install [pandoc](https://pandoc.org/)."
- [readme] This requires the devtools package, you can install it with install.packages("devtools").: "This requires the `devtools` package, you can install it with `install.packages("devtools")`."
