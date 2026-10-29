---
name: champ-pipeline-execution
description: Use when raw DNA methylation array IDAT files (450K/EPIC, e.g. from GEO
  accession GSE207426) must be processed through ChAMP to call differentially methylated
  CpGs (DMCs) and validate counts against a published analysis.
when_to_use_negative:
- Input is already a processed differential methylation results table — use methylR's
  downstream modules (Volcano plot, GO, pathway enrichment) instead
- Input is normalized beta values only (e.g. testDataFile2) — ChAMP loading/QC/normalization
  from raw IDATs cannot be run
- Host runs ARM64 (e.g. Apple Silicon) — methylR does not support the ARM64 chipset
  architecture
edam_topics:
- http://edamontology.org/topic_0182
tools:
- name: methylR
  role: Shiny GUI wrapper that launches and drives the ChAMP analysis module on uploaded
    raw methylation array data
  repo: https://github.com/JD2112/methylr
- name: ChAMP
  role: Recommended analysis pipeline inside methylR performing loading, quality control,
    normalization, and differential methylation calling
- name: Docker
  role: Container runtime used to run methylR locally on AMD64 hosts
- name: Singularity
  role: Alternative container runtime for HPC/server execution of the methylR container
    from the terminal
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:f790bd9f7c994424106024a8d8964c52677315a0064d527356687536a0f96034
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JD2112/methylr
  tools:
  - ChAMP
  - Docker
  - Singularity
  - methylR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0182
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# champ-pipeline-execution

## Summary

Executes the ChAMP differential methylation pipeline (loading, quality control, normalization, differential methylation calling) through methylR's Shiny interface to reproduce published DMC counts from Illumina 450K/EPIC methylation array data such as GSE207426.

## When to use

Use when raw DNA methylation array IDAT files (450K/EPIC, e.g. from GEO accession GSE207426) must be processed through ChAMP to call differentially methylated CpGs (DMCs) and validate counts against a published analysis.

## When NOT to use

- Input is already a processed differential methylation results table — use methylR's downstream modules (Volcano plot, GO, pathway enrichment) instead
- Input is normalized beta values only (e.g. testDataFile2) — ChAMP loading/QC/normalization from raw IDATs cannot be run
- Host runs ARM64 (e.g. Apple Silicon) — methylR does not support the ARM64 chipset architecture

## Inputs

- Raw IDAT files for a GEO methylation array dataset (e.g. GSE207426, Illumina 450K/EPIC)
- Sample sheet (phenotype/group metadata) accompanying the IDAT files
- methylR Docker container image (jd21/methylr:latest, AMD64)

## Outputs

- Differential methylation probe results table for the full dataset (DMC list with per-probe statistics)
- Total DMC count used for validation against the published figure (e.g. 123,510 DMCs for GSE207426)

## How to apply

Download the raw IDAT files and sample sheet for the target GEO dataset (e.g. GSE207426) and launch the methylR Docker container locally on an AMD64 Linux system with `docker run --rm -p 3838:3838 jd21/methylr:latest`, then open http://localhost:3838. Load the IDAT files plus sample sheet into the methylR Shiny interface and run the ChAMP module (the recommended pipeline), which performs the standard ChAMP chain of loading, quality control filtering, normalization, and differential methylation calling between sample groups. Export the resulting differential methylation probe results table for the full dataset. Validate reproduction by confirming the total number of DMCs equals the published count (123,510 for full-dataset GSE207426). If running Singularity/Apptainer instead of Docker, generate the shiny-server configuration with prepare_template.sh and bind the log/lib/conf paths as documented.

## Related tools

- **methylR** (Shiny GUI wrapper that launches and drives the ChAMP analysis module on uploaded raw methylation array data) — https://github.com/JD2112/methylr
- **ChAMP** (Recommended analysis pipeline inside methylR performing loading, quality control, normalization, and differential methylation calling)
- **Docker** (Container runtime used to run methylR locally on AMD64 hosts)
- **Singularity** (Alternative container runtime for HPC/server execution of the methylR container from the terminal)

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest
```

## Evaluation signals

- Total number of differentially methylated CpGs in the full-dataset results equals the published count of 123,510 DMCs
- All standard ChAMP steps (loading, quality control, normalization, differential methylation calling) complete without errors in the Shiny run
- Exported differential methylation probe results table is populated with per-probe statistics for the full dataset
- Browser session is closed and restarted between complete analyses to clear temporary memory, avoiding carry-over artifacts

## Limitations

- methylR is not supported on the ARM64 chipset architecture; AMD64 (Intel) Mac, Linux, and Windows (with WSL2 Ubuntu 20.04LTS) are required for the Docker container
- After one complete analysis (with ChAMP or minfi), the browser must be closed to clear temporary memory before starting again
- Reproduction is tied to ChAMP defaults within methylR; parameter changes versus the published run may change the DMC count

## Evidence

- [methods] ChAMP is the recommended pipeline module in methylR: "after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again."
- [methods] methylR is delivered as a Docker container for local execution: "*methylR* is packed into docker container that is available online."
- [methods] ARM64 architecture is unsupported for methylR: "NOTE: the ARM64 chipset architecture is actually not supported by methylR."
- [abstract] methylR reproduces published results on a public dataset: "We evaluated a publicly available dataset and compared the published results with those obtained with our tool"
- [readme] Run command for the methylR container: [evidence span withheld — anchor status: external_doc]
- [readme] methylR analyzes raw 450K and EPIC array data end to end: [evidence span withheld — anchor status: external_doc]
