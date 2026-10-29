---
name: docker-containerized-pipeline-execution
description: Reference for executing a Shiny/GUI bioinformatics pipeline locally via
  its published Docker (or Singularity/Apptainer) image when reproducing authors'
  analyses or benchmarking runtimes without installing R dependencies.
when_to_use_negative:
- Host is ARM64 (e.g. Apple Silicon) — methylR explicitly does not support the ARM64
  chipset architecture
- A non-containerized R environment with ChAMP/minfi already installed is required,
  e.g. for scripted batch runs outside the Shiny GUI
- The task needs raw-data processing of sequencing (non-array) methylation data; methylR
  targets 450K/EPIC Illumina arrays
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3518
tools:
- name: methylR
  role: Shiny web application encapsulating the full 450K/EPIC methylation analysis
    pipeline executed inside the container
  repo: https://github.com/JD2112/methylr
- name: Docker
  role: Container runtime used to launch methylR locally with port mapping 3838:3838
    (image jd21/methylr:latest, AMD64 only)
  repo: https://github.com/JD2112/methylr-devel
- name: Singularity
  role: Alternative container runtime for HPC clusters, running docker://jd21/methylr:latest
    with shiny-server bind mounts
- name: ChAMP
  role: Upstream analysis pipeline selectable inside methylR for a complete containerized
    methylation analysis
- name: minfi
  role: Alternative analysis pipeline selectable inside methylR, run in a fresh browser
    session to avoid memory carry-over
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:c05222160420f4c099e1ca57f824e8b624578cf83164272923d3672eb809b162
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JD2112/methylr
  tools:
  - ChAMP
  - Docker
  - R
  - Singularity
  - methylR
  - minfi
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# docker-containerized-pipeline-execution

## Summary

Run the methylR Shiny-based DNA methylation analysis pipeline (ChAMP or minfi) inside its prebuilt Docker container, so that raw 450K/EPIC Illumina array analysis through downstream modules executes in a reproducible, dependency-free local environment.

## When to use

Reference for executing a Shiny/GUI bioinformatics pipeline locally via its published Docker (or Singularity/Apptainer) image when reproducing authors' analyses or benchmarking runtimes without installing R dependencies.

## When NOT to use

- Host is ARM64 (e.g. Apple Silicon) — methylR explicitly does not support the ARM64 chipset architecture
- A non-containerized R environment with ChAMP/minfi already installed is required, e.g. for scripted batch runs outside the Shiny GUI
- The task needs raw-data processing of sequencing (non-array) methylation data; methylR targets 450K/EPIC Illumina arrays

## Inputs

- Raw Illumina DNA methylation array data (450K or EPIC) loaded into the methylR web UI at http://localhost:3838
- methylR testData.zip from https://sourceforge.net/projects/methylr/files/testData.zip
- Prebuilt Docker image jd21/methylr:latest (AMD64 architecture)

## Outputs

- Complete ChAMP or minfi methylation analysis results delivered through the Shiny interface (normalization, DMCs, downstream modules)
- Wall-clock runtime measurements per pipeline (e.g. ~3 min ChAMP, ~4 min minfi on test data)
- Shiny server logs (e.g. /tmp/shiny-server.* when run under Singularity)

## How to apply

Pull the prebuilt image and launch it with `docker run --rm -p 3838:3838 jd21/methylr:latest`, then open http://localhost:3838 in a supported browser (e.g. Firefox 105 on Ubuntu 20.04 LTS). Confirm the host is AMD64 — ARM64 is explicitly unsupported — and note the container must be run with a single core and ~12 GiB RAM to match the paper's minimal hardware claims. For clusters where Docker is unavailable, convert and run the same image via Singularity/Apptainer using the prepare_template.sh bindings described in the README (tested with Singularity v3.8.6). After one complete analysis with either ChAMP or minfi, close the browser to clear temporary memory and restart the session before the next run, preventing memory carry-over between pipelines. Download the testData.zip methylation array dataset from SourceForge to exercise the pipeline end-to-end.

## Related tools

- **methylR** (Shiny web application encapsulating the full 450K/EPIC methylation analysis pipeline executed inside the container) — https://github.com/JD2112/methylr
- **Docker** (Container runtime used to launch methylR locally with port mapping 3838:3838 (image jd21/methylr:latest, AMD64 only)) — https://github.com/JD2112/methylr-devel
- **Singularity** (Alternative container runtime for HPC clusters, running docker://jd21/methylr:latest with shiny-server bind mounts)
- **ChAMP** (Upstream analysis pipeline selectable inside methylR for a complete containerized methylation analysis)
- **minfi** (Alternative analysis pipeline selectable inside methylR, run in a fresh browser session to avoid memory carry-over)

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest
```

## Evaluation signals

- Shiny server startup log appears (e.g. 'Starting listener on http://[::]:15910' under Singularity) and http://localhost:3838 renders the methylR UI in the browser
- A complete ChAMP run on testData.zip finishes in approximately 3 minutes and minfi in approximately 4 minutes on 1 core / 12 GiB RAM
- Container starts with the documented command `docker run --rm -p 3838:3838 jd21/methylr:latest` without architecture errors on an AMD64 host
- Closing the browser between ChAMP and minfi runs clears temporary memory; identical inputs then produce consistent results across sessions
- Downstream modules (GO/pathway enrichment) accept the DMC file produced by the containerized run, confirming the analysis completed end-to-end

## Limitations

- ARM64 chipset architecture is not supported; the Docker image is built for AMD64 only
- Only one complete analysis should be run per browser session — the browser must be closed to clear temporary memory before starting again
- Web server endpoint moved from methylr.research.liu.se to https://methylr.serve.scilifelab.se/, so published web URLs may be stale

## Evidence

- [methods] methylR is packed into docker container that is available online: "*methylR* is packed into docker container that is available online."
- [methods] close the browser after one complete analysis: "after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again."
- [methods] ARM64 not supported: "NOTE: the ARM64 chipset architecture is actually not supported by methylR."
- [readme] docker run command from README: [evidence span withheld — anchor status: external_doc]
- [methods] test data download location: "If you want to run with the test data, Please download the testdata from [https://sourceforge.net/projects/methylr/files/testData.zip]"
