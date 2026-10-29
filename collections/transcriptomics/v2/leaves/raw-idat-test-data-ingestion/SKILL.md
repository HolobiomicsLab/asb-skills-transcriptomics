---
name: raw-idat-test-data-ingestion
description: Use when reproducing or smoke-testing the methylR pipeline on the authors'
  distributed test dataset, or when Illumina 450K/EPIC raw methylation data (IDAT)
  must be ingested into a locally running methylR container.
when_to_use_negative:
- You only need the accessory GUI demo inputs (Volcano, PCA/MDS, heatmap, pathway
  enrichment) — those are in the GitHub data/ directory, not testData.zip
- Your host is ARM64, which is explicitly not supported by methylR
- You need to analyze your own 450K/EPIC samples rather than reproduce the authors'
  test-data workflow
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3518
tools:
- name: methylR
  role: Shiny-based GUI pipeline into which the test data is loaded and the ChAMP/minfi
    analysis is executed
  repo: https://github.com/JD2112/methylr
- name: Docker
  role: Container runtime used to launch methylR locally (AMD64 architecture only)
  repo: https://github.com/JD2112/methylr-devel
- name: ChAMP
  role: One of the two methylation analysis pipelines executed on the ingested test
    data
- name: minfi
  role: Alternative methylation analysis pipeline executed on the ingested test data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:6be37eaf6d57b276f26f07b97cb5b31c71afc0b3f4790c6bd191db30559ad78d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JD2112/methylr
  tools:
  - ChAMP
  - Docker
  - methylR
  - minfi
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# raw-idat-test-data-ingestion

## Summary

Obtains and loads methylR's Illumina methylation array test dataset (testData.zip from SourceForge) into the locally-run methylR Shiny/Docker container so that complete ChAMP or minfi analyses can be executed on raw data with a few mouse clicks.

## When to use

Use when reproducing or smoke-testing the methylR pipeline on the authors' distributed test dataset, or when Illumina 450K/EPIC raw methylation data (IDAT) must be ingested into a locally running methylR container.

## When NOT to use

- You only need the accessory GUI demo inputs (Volcano, PCA/MDS, heatmap, pathway enrichment) — those are in the GitHub data/ directory, not testData.zip
- Your host is ARM64, which is explicitly not supported by methylR
- You need to analyze your own 450K/EPIC samples rather than reproduce the authors' test-data workflow

## Inputs

- testData.zip (Illumina DNA methylation array raw test data from SourceForge)
- testDataFile1 (differentially methylated data file for gene features, Volcano plot, chromosome map, GO and pathway enrichment modules)
- testDataFile2 (normalized beta value table for MDS/PCA plots)
- heatmapMatrix test data file

## Outputs

- A loaded methylR session ready to run a complete ChAMP or minfi analysis on the test dataset
- Downstream methylR outputs from the eight analysis modules (normalization, DMCs, plots, pathway enrichment) when a full run is executed

## How to apply

Download testData.zip from https://sourceforge.net/projects/methylr/files/testData.zip and unpack it — this archive holds the DNA methylation raw files, while accessory test files (testDataFile1 for DMR/Volcano/chromosome map/GO/pathway modules, testDataFile2 for MDS/PCA, heatmapMatrix for the heatmap module) live in the GitHub data directory. Launch the methylR container with `docker run --rm -p 3838:3838 jd21/methylr:latest` and open http://localhost:3838 in a supported browser (e.g. Firefox 105 on Ubuntu 20.04LTS). Load the unpacked test data through the Shiny GUI and execute one complete analysis with either the ChAMP or minfi pipeline. After one complete analysis, close the browser to clear temporary memory and restart before a second run so memory carry-over does not bias subsequent results.

## Related tools

- **methylR** (Shiny-based GUI pipeline into which the test data is loaded and the ChAMP/minfi analysis is executed) — https://github.com/JD2112/methylr
- **Docker** (Container runtime used to launch methylR locally (AMD64 architecture only)) — https://github.com/JD2112/methylr-devel
- **ChAMP** (One of the two methylation analysis pipelines executed on the ingested test data)
- **minfi** (Alternative methylation analysis pipeline executed on the ingested test data)

## Examples

```
curl -L -o testData.zip https://sourceforge.net/projects/methylr/files/testData.zip && unzip testData.zip && docker run --rm -p 3838:3838 jd21/methylr:latest
```

## Evaluation signals

- testData.zip unpacks and the methylation raw files load into the methylR GUI without errors
- A complete analysis runs end-to-end with the ChAMP pipeline (~3 min wall-clock) and separately with minfi (~4 min wall-clock) on minimal hardware (1 core, 12 GiB RAM)
- Downstream module outputs (e.g. DMC tables, MDS/PCA plots) are produced from the loaded test data
- A second pipeline run after closing and restarting the browser reproduces the first run's timing, confirming no memory carry-over

## Limitations

- ARM64 chipset architecture is not supported by methylR; only AMD64 builds are functional
- The Docker container must be run on supported OS configurations (e.g. Ubuntu 20.04LTS with Docker 20.10.18, or Windows 10 21H2 with WSL2)
- The browser must be closed between complete analyses to clear temporary memory and start fresh

## Evidence

- [readme] DNA methylation test data can be found [here](https://sourceforge.net/projects/methylr/files/testData.zip): [evidence span withheld — anchor status: external_doc]
- [methods] methylR is packed into docker container that is available online.: "*methylR* is packed into docker container that is available online."
- [methods] close the browser (to clear the temporary memory) and start again.: "after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again."
- [readme] the ARM64 chipset architecture is actually not supported by methylR.: "**PLEASE NOTE: the ARM64 chipset architecture is actually not supported by *methylR*.**"
