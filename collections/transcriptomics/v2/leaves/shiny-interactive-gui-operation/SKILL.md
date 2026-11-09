---
name: shiny-interactive-gui-operation
description: Use when a practitioner needs to execute differential methylation analysis
  and its downstream feature modules (volcano plot, chromosome map, PCA/MDS, gene
  ontology, pathway enrichment) through an interactive GUI rather than writing R code.
when_to_use_negative:
- Running on ARM64 hardware — the Docker container is built on AMD64 architecture
  and methylR explicitly does not support ARM64 chipsets
- Headless, scripted or reproducible batch pipelines where a non-interactive CLI workflow
  is preferable to a mouse-driven Shiny GUI
- Analysis targets beyond DNA methylation arrays that require the primary methylation
  pipeline rather than methylR's accessory downstream modules
edam_operation: http://edamontology.org/operation_0337
tools:
- name: methylR
  role: The Shiny-based graphical interface whose modules (Feature Analysis, volcano
    plot, etc.) are operated interactively through the browser
  repo: https://github.com/JD2112/methylr
- name: Docker
  role: Containerization that packages methylR and Shiny/R for local deployment with
    'docker run --rm -p 3838:3838 jd21/methylr:latest'
  repo: https://github.com/JD2112/methylr-devel
- name: Singularity
  role: Alternative container runtime (v3.8.6 tested) for running the methylr Docker
    image on HPC via singularity run with shiny-server bind mounts
- name: shiny
  role: R web framework that implements the interactive GUI driving the methylysis
    and Feature Analysis modules
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:21da1e28f36ab157ee36932b9bbc305691324b08d0a0b0e74b88371cf0375a2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JD2112/methylr
  tools:
  - Docker
  - Singularity
  - methylR
  - shiny
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# shiny-interactive-gui-operation

## Summary

Runs a Shiny-based graphical interface (methylR) inside a Docker container to perform DNA methylation array analysis — from raw 450K/EPIC data through downstream modules like volcano plots, gene ontology and pathway enrichment — entirely through mouse clicks instead of scripted pipelines.

## When to use

Use when a practitioner needs to execute differential methylation analysis and its downstream feature modules (volcano plot, chromosome map, PCA/MDS, gene ontology, pathway enrichment) through an interactive GUI rather than writing R code.

## When NOT to use

- Running on ARM64 hardware — the Docker container is built on AMD64 architecture and methylR explicitly does not support ARM64 chipsets
- Headless, scripted or reproducible batch pipelines where a non-interactive CLI workflow is preferable to a mouse-driven Shiny GUI
- Analysis targets beyond DNA methylation arrays that require the primary methylation pipeline rather than methylR's accessory downstream modules

## Inputs

- DNA methylation raw files from Illumina 450K or EPIC arrays (testData.zip from SourceForge)
- Differentially methylated CpG (DMC) data file produced by the ChAMP- or minfi-based methylysis step (e.g. testDataFile1)
- Normalized beta value data table for MDS/PCA modules (e.g. testDataFile2)
- Group data information ('groupData') for PCA plots
- User-adjusted adjusted-P-value and log-fold-change cutoffs entered in the GUI

## Outputs

- Interactive Shiny visualizations rendered in the browser (volcano plot of -log10(adjusted P-value) versus logFC, MDS, PCA, heatmap, chromosome map)
- Exported list of significant CpG IDs passing both the adjusted-P and logFC cutoffs
- Downstream functional and pathway enrichment results

## How to apply

Launch the methylR Shiny web server from a Docker container on AMD64 hardware (e.g. 'docker run --rm -p 3838:3838 jd21/methylr:latest') and open http://localhost:3838 in a supported browser (Firefox, Chrome, Safari, Edge at documented versions). In the GUI, run one complete analysis — either with ChAMP or minfi as the differential analysis backend — then use the Feature Analysis modules by uploading the DMC results or normalized beta value tables. For the volcano plot module, interactively set the adjusted P-value and logFC cutoffs; the Shiny app re-renders the plot and regenerates the significant CpG list with each change. After one complete analysis, close the browser to clear Shiny's temporary memory before starting a new run, since state persists in the browser session.

## Related tools

- **methylR** (The Shiny-based graphical interface whose modules (Feature Analysis, volcano plot, etc.) are operated interactively through the browser) — https://github.com/JD2112/methylr
- **Docker** (Containerization that packages methylR and Shiny/R for local deployment with 'docker run --rm -p 3838:3838 jd21/methylr:latest') — https://github.com/JD2112/methylr-devel
- **Singularity** (Alternative container runtime (v3.8.6 tested) for running the methylr Docker image on HPC via singularity run with shiny-server bind mounts)
- **shiny** (R web framework that implements the interactive GUI driving the methylysis and Feature Analysis modules)

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest
```

## Evaluation signals

- The Shiny server starts successfully and http://localhost:3838 (or the Singularity-bound port, e.g. 15910) serves the methylR GUI in a supported browser
- The rendered volcano plot and exported significant CpG list are mutually consistent: every CpG in the exported list lies beyond both the adjusted-P-value and logFC cutoffs shown on the plot
- A complete ChAMP- or minfi-based analysis runs end-to-end inside the browser session without errors
- The browser is closed and temporary memory cleared after one complete analysis before starting another, so results from the previous run do not leak into the next

## Limitations

- The ARM64 chipset architecture is not supported by methylR; the Docker container was built on AMD64 OS architecture to get full functional capabilities of Shiny/R
- After one complete analysis (ChAMP or minfi), the browser must be closed to clear temporary memory before starting again
- Docker version and browser versions matter: tested with Docker 20.10.17–20.10.20 and specific Firefox/Chrome/Safari/Edge versions on Ubuntu 20.04, macOS Monterey (Intel), and Windows 10 with WSL2
- Running Singularity/Apptainer containers required a workaround script (prepare_template.sh) and specific bind mounts; vanilla singularity run initially failed

## Evidence

- [methods] Here, in *methylR*, we presented a shiny-based web server approach to minimize the above-mentioned difficulties.: "Here, in *methylR*, we presented a shiny-based web server approach to minimize the above-mentioned difficulties."
- [methods] *methylR* is packed into docker container that is available online.: "*methylR* is packed into docker container that is available online."
- [methods] after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again.: "after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again."
- [readme] the docker container was built on AMD64 OS architecture to get the full functional capabilities of Shiny/R.: [evidence span withheld — anchor status: external_doc]
- [readme] PLEASE NOTE: the ARM64 chipset architecture is actually not supported by *methylR*.: "PLEASE NOTE: the ARM64 chipset architecture is actually not supported by *methylR*."
- [readme] testDataFile1 - differentially methylated data file, can be used for gene features, Volcano plot, chromosome map, gene ontology and pathway enrichment analysis modules.: [evidence span withheld — anchor status: not_found]
