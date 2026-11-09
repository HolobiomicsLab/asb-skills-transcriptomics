---
name: docker-containerized-bioinformatics-workflow
description: Use when you need to execute methylR locally on raw Illumina 450K/EPIC
  methylation array data (IDAT plus sample sheet) via its Docker image, e.g. to reproduce
  published differential methylation results such as DMC counts on a GEO dataset.
when_to_use_negative:
- You are running on ARM64 hardware — the README states the ARM64 chipset architecture
  is not supported by methylR (container is built for AMD64).
- Your data are not Illumina 450K/EPIC methylation arrays (e.g. WGBS or RNA-seq) —
  the core ChAMP/minfi analysis modules target array methylation data.
- You need command-line, headless pipeline automation rather than interactive GUI-driven
  analysis — methylR is a Shiny web interface, not a batch CLI.
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3518
tools:
- name: methylR
  role: Shiny-based GUI pipeline that wraps ChAMP/minfi for end-to-end methylation
    array analysis; delivered as the Docker image being run
  repo: https://github.com/JD2112/methylr
- name: ChAMP
  role: Recommended analysis module inside methylR performing loading, QC, normalization,
    and differential methylation calling
- name: minfi
  role: Alternative analysis backend selectable within methylR instead of ChAMP
- name: Docker
  role: Container runtime used to launch methylR locally with port 3838 mapped to
    the Shiny server
  repo: https://github.com/JD2112/methylr-devel
- name: Singularity
  role: Alternative container runtime for running the same Docker image from the terminal
    (e.g. HPC), using prepare_template.sh to generate Shiny server configuration and
    bind mounts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:b92f4c7dc6cfdaf00030147beb2a5848bd8473e913f27d009db4eeed0a9eb672
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JD2112/methylr
  tools:
  - ChAMP
  - Docker
  - Singularity
  - methylR
  - minfi
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# docker-containerized-bioinformatics-workflow

## Summary

Run the methylR Shiny-based DNA methylation analysis pipeline as a locally launched Docker container (AMD64 only), so that raw 450K/EPIC IDAT data can be processed with ChAMP or minfi through a browser GUI without local R installation. This enables reproducible, containerized execution of the full methylation workflow from sequencer data to pathway analysis.

## When to use

Use when you need to execute methylR locally on raw Illumina 450K/EPIC methylation array data (IDAT plus sample sheet) via its Docker image, e.g. to reproduce published differential methylation results such as DMC counts on a GEO dataset.

## When NOT to use

- You are running on ARM64 hardware — the README states the ARM64 chipset architecture is not supported by methylR (container is built for AMD64).
- Your data are not Illumina 450K/EPIC methylation arrays (e.g. WGBS or RNA-seq) — the core ChAMP/minfi analysis modules target array methylation data.
- You need command-line, headless pipeline automation rather than interactive GUI-driven analysis — methylR is a Shiny web interface, not a batch CLI.

## Inputs

- Raw Illumina 450K/EPIC DNA methylation array IDAT files
- Sample sheet (phenotype/group metadata) matching the IDAT files
- Optional methylR test data (testData.zip from SourceForge; testDataFile1, testDataFile2, heatmapMatrix from the GitHub data directory)

## Outputs

- Differentially methylated probe (DMP/DMC) results table exported from the methylR Shiny interface
- Normalized beta-value matrices and downstream module outputs (Volcano plots, chromosome maps, GO and pathway enrichment, MDS/PCA plots, heatmaps)

## How to apply

Pull and run the published methylR Docker image (jd21/methylr:latest) on an AMD64 host, exposing port 3838 with `docker run --rm -p 3838:3838 jd21/methylr:latest`, then open http://localhost:3838 in a supported browser (e.g. Firefox 105 on Ubuntu 20.04 LTS). The container bundles the Shiny server, R, and the analysis backends (ChAMP and minfi). Load the dataset (e.g. IDAT files plus sample sheet from GEO) into the Shiny interface, choose the ChAMP module (the recommended pipeline covering loading, quality control, normalization, and differential methylation calling), and export the DMP results table. Close the browser after each complete analysis to clear temporary memory before starting a new one. Where Docker is unavailable (e.g. HPC clusters), the same container can be run via Singularity/Apptainer using the prepare_template.sh helper to generate shiny-server.conf bindings.

## Related tools

- **methylR** (Shiny-based GUI pipeline that wraps ChAMP/minfi for end-to-end methylation array analysis; delivered as the Docker image being run) — https://github.com/JD2112/methylr
- **ChAMP** (Recommended analysis module inside methylR performing loading, QC, normalization, and differential methylation calling)
- **minfi** (Alternative analysis backend selectable within methylR instead of ChAMP)
- **Docker** (Container runtime used to launch methylR locally with port 3838 mapped to the Shiny server) — https://github.com/JD2112/methylr-devel
- **Singularity** (Alternative container runtime for running the same Docker image from the terminal (e.g. HPC), using prepare_template.sh to generate Shiny server configuration and bind mounts)

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest
```

## Evaluation signals

- The Shiny server starts successfully and http://localhost:3838 serves the methylR interface in a supported AMD64-compatible browser.
- A complete run on a reference dataset reproduces published results — e.g. GSE207426 full-dataset analysis yields exactly 123,510 DMCs as in the published analysis.
- The exported DMP results table is populated with probe-level differential methylation statistics for the full dataset.
- Successive analyses are run in a fresh browser session (browser closed between runs) to avoid stale in-memory state from the prior ChAMP/minfi run.
- Singularity execution (if used) shows the Shiny Server startup log lines ('Starting listener on http://[::]:15910') and the app is reachable at the bound port.

## Limitations

- ARM64 chipset architecture is not supported; the Docker container was built on AMD64 OS architecture for full Shiny/R functionality.
- Docker images are version-sensitive: README-tested versions include Docker 20.10.17–20.10.20 and Docker Desktop 4.12.0–4.13.0 on Ubuntu 20.04 LTS, macOS Monterey 12.5.1 (Intel), and Windows 10 21H2 with WSL2.
- Only one complete ChAMP or minfi analysis can occupy the app's temporary memory at a time; the browser must be closed to clear it before restarting.
- Singularity/Apptainer runs require helper script (prepare_template.sh) to configure shiny-server binds; the port (e.g. 15910) differs from the Docker default 3838.
- The methylR web server has moved from methylr.research.liu.se to https://methylr.serve.scilifelab.se/ — local container use avoids dependence on the hosted service.

## Evidence

- [methods] methylR is packed into docker container that is available online.: "*methylR* is packed into docker container that is available online."
- [readme] ARM64 not supported: "**PLEASE NOTE: the ARM64 chipset architecture is actually not supported by *methylR*.**"
- [readme] docker run command: [evidence span withheld — anchor status: external_doc]
- [methods] close browser between analyses: "after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again."
- [readme] web server moved: "**IMPORTANT: MethylR web-server moved from methylr.research.liu.se to [https://methylr.serve.scilifelab.se/](https://methylr.serve.scilifelab.se/)**"
