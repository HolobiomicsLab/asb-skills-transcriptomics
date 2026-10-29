---
name: shiny-gui-pipeline-execution
description: Use when you need to execute methylR locally via its containerized Shiny
  GUI for Illumina 450K/EPIC array analysis, or when reproducing the paper's downstream
  modules (DMC plots, GO, KEGG/Reactome/WikiPathways enrichment) without writing R
  code.
when_to_use_negative:
- Host hardware is ARM64 chipset architecture — methylR explicitly does not support
  it
- Goal is headless/scriptable non-interactive batch processing — the skill is defined
  by mouse-driven Shiny GUI use, not a CLI API
- Data are not Illumina 450K/EPIC DNA methylation arrays (e.g., sequencing-based methylation),
  unless using the accessory modules the authors made suitable for other kinds of
  data analysis
tools:
- name: methylR
  role: The Shiny-based graphical pipeline being executed; provides all analysis modules
    from raw array data to pathway enrichment
  repo: https://github.com/JD2112/methylr
- name: Docker
  role: Container runtime packaging the Shiny app for local execution (`docker run
    --rm -p 3838:3838 jd21/methylr:latest`)
  repo: https://github.com/JD2112/methylr-devel/actions/workflows/docker-image.yml
- name: Singularity
  role: Alternative container runtime for running the methylR Docker image directly
    from the terminal (e.g., on clusters), tested with Singularity v3.8.6
- name: shiny
  role: R web-framework underlying the methylr GUI; the single interface from sequencer
    data to pathway analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:8f38b12b20f98a51bb473b06e3fa58708a7cb324dc242c844e312307f409dce9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JD2112/methylr
  tools:
  - Docker
  - Singularity
  - methylR
  - shiny
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# shiny-gui-pipeline-execution

## Summary

Running the methylR Shiny-based graphical pipeline inside its packaged Docker (or Singularity) container so that raw DNA methylation array analysis through pathway enrichment can be executed end-to-end with mouse clicks in a web browser.

## When to use

Use when you need to execute methylR locally via its containerized Shiny GUI for Illumina 450K/EPIC array analysis, or when reproducing the paper's downstream modules (DMC plots, GO, KEGG/Reactome/WikiPathways enrichment) without writing R code.

## When NOT to use

- Host hardware is ARM64 chipset architecture — methylR explicitly does not support it
- Goal is headless/scriptable non-interactive batch processing — the skill is defined by mouse-driven Shiny GUI use, not a CLI API
- Data are not Illumina 450K/EPIC DNA methylation arrays (e.g., sequencing-based methylation), unless using the accessory modules the authors made suitable for other kinds of data analysis

## Inputs

- testData.zip (DNA methylation test data) from https://sourceforge.net/projects/methylr/files/testData.zip
- testDataFile1 — differentially methylated data file (for gene features, Volcano plot, chromosome map, GO and pathway enrichment modules)
- testDataFile2 — normalized beta value data table (for MDS/PCA plots, PCA requiring 'groupData')
- heatmapMatrix test data file
- User-supplied Illumina 450K/EPIC raw methylation array data

## Outputs

- Interactive GUI results per module (volcano plots, chromosome maps, PCA/MDS plots, heatmaps)
- Functional (GO) and pathway enrichment results tables (KEGG/Reactome/WikiPathways)
- Annotated genomic feature outputs from DMC analysis

## How to apply

Launch the container with `docker run --rm -p 3838:3838 jd21/methylr:latest` on an AMD64 Linux/Mac/Windows host (the image is built on AMD64; ARM64 is explicitly unsupported), then open a supported browser (Firefox 105, Chrome 106/107, Edge 106, or Safari 15.6.1) at http://localhost:3838. For HPC/cluster contexts use Singularity/Apptainer via the prepare_template.sh helper from singularity-shiny, binding the generated shiny-server.conf, logs, and lib directories. Load input data (testData.zip from SourceForge, or testDataFile1 for DMC/gene-features/pathway modules and testDataFile2 for MDS/PCA) through the GUI and run the eight downstream analysis modules, including ChAMP- or minfi-based analysis. After one complete analysis (with ChAMP or minfi), close the browser to clear temporary memory before starting a new run — this is a required state-reset step of the Shiny workflow.

## Related tools

- **methylR** (The Shiny-based graphical pipeline being executed; provides all analysis modules from raw array data to pathway enrichment) — https://github.com/JD2112/methylr
- **Docker** (Container runtime packaging the Shiny app for local execution (`docker run --rm -p 3838:3838 jd21/methylr:latest`)) — https://github.com/JD2112/methylr-devel/actions/workflows/docker-image.yml
- **Singularity** (Alternative container runtime for running the methylR Docker image directly from the terminal (e.g., on clusters), tested with Singularity v3.8.6)
- **shiny** (R web-framework underlying the methylr GUI; the single interface from sequencer data to pathway analysis)

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest
```

## Evaluation signals

- Browser reaches the Shiny app at http://localhost:3838 (Docker) or localhost:15910 (Singularity) and the GUI loads without errors
- A complete ChAMP- or minfi-based analysis finishes; per the paper, the browser must then be closed and reopened to clear temporary memory before a new run
- testDataFile1 loads and drives the gene features, Volcano plot, chromosome map, gene ontology and pathway enrichment modules; testDataFile2 drives MDS/PCA (PCA additionally needing groupData)
- Docker/Singularity run logs show shiny-server startup lines ('Starting listener on http://...') confirming the server booted with the generated shiny-server.conf
- Published-results comparison: methylR outputs on the publicly available dataset match those reported in the original study

## Limitations

- ARM64 chipset architecture is not supported; the Docker image was built on AMD64 only
- Docker/Singularity execution on MacOS/Windows AMD64 follows separate manual instructions (methylr.netlify.app/dockercontainer.html)
- Closing the browser is required after each complete ChAMP/minfi analysis to clear temporary memory — skipping it can corrupt a subsequent run
- No changelog is available in the repository, so version drift is hard to track

## Evidence

- [methods] methylR is packed into docker container that is available online.: "*methylR* is packed into docker container that is available online."
- [methods] Singularity runs the docker image from terminal: "Singularity can also be used to run the docker container directly from terminal."
- [methods] Close browser to clear temporary memory after one complete analysis: "after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again."
- [methods] ARM64 not supported: "NOTE: the ARM64 chipset architecture is actually not supported by methylR."
- [readme] Docker run command from README: [evidence span withheld — anchor status: external_doc]
- [readme] Test data file roles from README: [evidence span withheld — anchor status: not_found]
- [readme] Web-server relocation note from README: "IMPORTANT: MethylR web-server moved from methylr.research.liu.se to [https://methylr.serve.scilifelab.se/](https://methylr.serve.scilifelab.se/)"
