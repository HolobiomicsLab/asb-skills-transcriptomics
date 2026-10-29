---
name: dna-methylation-array-data-processing
description: Use when raw Illumina 450K or EPIC methylation array files (IDAT plus
  sample sheet) need loading, QC, normalization, and differential methylation analysis,
  or when reproducing a published DMC analysis.
when_to_use_negative:
- The host CPU is ARM64 — the README states 'the ARM64 chipset architecture is actually
  not supported by methylR'
- Input is RNA-seq or other sequencing data; methylR is scoped to Illumina 450K and
  EPIC methylation arrays (though accessory modules may handle other tabular data
  types)
- Only an interactive web result is needed and no local container can be run on an
  AMD64 Linux/Mac(Intel)/Windows host
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: methylR
  role: Shiny-based graphical pipeline that runs raw-data loading through downstream
    analyses with a few mouse clicks; packaged as a Docker container
  repo: https://github.com/JD2112/methylr
- name: ChAMP
  role: Recommended analysis backend inside methylR performing loading, QC, normalization,
    and differential methylation calling
- name: minfi
  role: Alternative analysis backend within methylR for the same complete analysis
- name: Docker
  role: Container runtime used to deploy methylR locally (AMD64 images only)
  repo: https://github.com/JD2112/methylr-devel
- name: Singularity
  role: Alternative container engine to run the methylR Docker image directly from
    the terminal (e.g. HPC clusters, tested with Singularity v3.8.6)
- name: shiny
  role: Web GUI framework exposing the methylR analysis modules
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:a59588ddf141a22faa8893e8b48ea16663655ec11875fdcf7c2807013ec37425
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
  - shiny
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# dna-methylation-array-data-processing

## Summary

End-to-end processing and analysis of Illumina 450K/EPIC DNA methylation array data (raw IDAT through DMC calling and downstream enrichment) via the methylR Shiny GUI running ChAMP or minfi pipelines in a Docker container.

## When to use

Use when raw Illumina 450K or EPIC methylation array files (IDAT plus sample sheet) need loading, QC, normalization, and differential methylation analysis, or when reproducing a published DMC analysis.

## When NOT to use

- The host CPU is ARM64 — the README states 'the ARM64 chipset architecture is actually not supported by methylR'
- Input is RNA-seq or other sequencing data; methylR is scoped to Illumina 450K and EPIC methylation arrays (though accessory modules may handle other tabular data types)
- Only an interactive web result is needed and no local container can be run on an AMD64 Linux/Mac(Intel)/Windows host

## Inputs

- Raw Illumina 450K/EPIC methylation array IDAT files plus sample sheet (e.g. GEO dataset GSE207426)
- testData.zip from https://sourceforge.net/projects/methylr/files/testData.zip
- Differentially methylated CpG list (DMC list, e.g. testDataFile1) for downstream modules
- Normalized beta value data table (testDataFile2) with optional 'groupData' for PCA
- Heatmap matrix test data file for the heatmap module

## Outputs

- Differential methylation probe results table (DMCs) for the full dataset
- Pathway enrichment results table (KEGG, Reactome, WikiPathways) matching the Figure 1e layout
- QC, normalization diagnostics, and visualizations (Volcano plot, chromosome map, MDS/PCA plots, heatmap)

## How to apply

Launch the methylR Docker container on an AMD64 host (Linux; ARM64 is unsupported) with `docker run --rm -p 3838:3838 jd21/methylr:latest` and open http://localhost:3838 in the browser. Load either raw methylation array data (IDAT files plus sample sheet, e.g. GSE207426 from GEO) or a precomputed DMC list / normalized beta table into the Shiny interface and run one complete analysis with the ChAMP pipeline (recommended, executing loading, quality control, normalization, and differential methylation calling) or the minfi alternative. After each complete analysis, close the browser to clear temporary memory and restart before a second run (e.g. switching from ChAMP to minfi) to prevent memory carry-over. For downstream modules, feed the DMC list into GO, pathway enrichment (KEGG, Reactome, WikiPathways), Volcano plot, chromosome map, and gene-feature annotation; use normalized beta values with group data for MDS/PCA plots. Export the differential methylation probe results table and validate, e.g. by confirming the total DMC count matches a published reference (123,510 DMCs for the full GSE207426 dataset) or that runtimes match ~3 min (ChAMP) and ~4 min (minfi) on 1 core / 12 GiB RAM.

## Related tools

- **methylR** (Shiny-based graphical pipeline that runs raw-data loading through downstream analyses with a few mouse clicks; packaged as a Docker container) — https://github.com/JD2112/methylr
- **ChAMP** (Recommended analysis backend inside methylR performing loading, QC, normalization, and differential methylation calling)
- **minfi** (Alternative analysis backend within methylR for the same complete analysis)
- **Docker** (Container runtime used to deploy methylR locally (AMD64 images only)) — https://github.com/JD2112/methylr-devel
- **Singularity** (Alternative container engine to run the methylR Docker image directly from the terminal (e.g. HPC clusters, tested with Singularity v3.8.6))
- **shiny** (Web GUI framework exposing the methylR analysis modules)

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest
```

## Evaluation signals

- Full-dataset DMC count from the exported probe results table equals the published reference (123,510 DMCs for GSE207426)
- ChAMP-pipeline wall-clock runtime on testData is ~3 minutes and minfi ~4 minutes on 1 core / 12 GiB RAM (within tolerance)
- Exported pathway enrichment table is a parseable tabular file whose layout matches the Figure 1e results table (per-database pathway entries with enrichment statistics)
- Analysis completes through the standard ChAMP steps (loading, quality control, normalization, differential methylation) without errors in the Shiny session
- Browser is closed and session restarted between consecutive complete analyses to avoid memory carry-over artifacts

## Limitations

- ARM64 chipset architecture is not supported by methylR; only AMD64 Docker images are provided
- Tested successfully on Linux-based systems (Ubuntu 20.04 LTS with Docker 20.10.18); Mac requires Intel/AMD64 (Monterey 12.5.1) and Windows requires WSL2 with Ubuntu 20.04 LTS
- Consecutive complete analyses require closing the browser to clear temporary memory; running ChAMP and minfi back-to-back without restart risks memory carry-over

## Evidence

- [abstract] methylR analyzes raw arrays via ChAMP/minfi in Shiny GUI: "we developed methylR a graphical tool that can analyze not only the raw data but also performs different downstream analyses with a few mouse clicks"
- [methods] Close browser between complete analyses: "after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again."
- [methods] ARM64 not supported: "NOTE: the ARM64 chipset architecture is actually not supported by methylR."
- [readme] Docker run command from README: [evidence span withheld — anchor status: external_doc]
- [results] ChAMP module steps performed on GSE207426: [evidence span withheld — anchor status: not_found]
