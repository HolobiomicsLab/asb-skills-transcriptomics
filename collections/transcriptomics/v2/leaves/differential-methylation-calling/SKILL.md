---
name: differential-methylation-calling
description: Use when raw IDAT files plus a sample sheet from an Illumina 450K/EPIC
  methylation array study (e.g. from GEO) need differential methylation calling between
  groups, and a GUI-driven, reproducible pipeline is acceptable.
when_to_use_negative:
- The host is ARM64 (e.g. Apple Silicon) — methylR's Docker container is built for
  AMD64 and ARM64 is not supported
- Data are already normalized beta-value tables or precomputed DMC lists — use methylR's
  downstream accessory modules (MDS/PCA, volcano, enrichment) instead of the ChAMP
  differential calling module
- The analysis requires command-line scripting or cluster HPC integration beyond what
  the Shiny GUI offers — consider running ChAMP or minfi directly in R
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: methylR
  role: Shiny-based GUI pipeline that packages ChAMP/minfi for raw methylation array
    loading, QC, normalization, and DMC calling
  repo: https://github.com/JD2112/methylr
- name: ChAMP
  role: Recommended analysis module inside methylR that performs loading, QC, normalization,
    and differential methylation calling
- name: Docker
  role: Container runtime used to launch methylR locally on AMD64 (docker run --rm
    -p 3838:3838 jd21/methylr:latest)
  repo: https://github.com/JD2112/methylr-devel
- name: minfi
  role: Alternative analysis module to ChAMP within methylR for the complete methylation
    analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:ca6b897425af6a2a261db00c66a85156eaa1840c08374bc2a1eb8b744891b340
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JD2112/methylr
  tools:
  - ChAMP
  - Docker
  - methylR
  - minfi
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# differential-methylation-calling

## Summary

Call differentially methylated CpGs (DMCs) from raw Illumina 450K/EPIC methylation array data using the ChAMP analysis module inside the methylR Shiny/Docker pipeline. Validated by reproducing the published full-dataset DMC count (123,510 DMCs) on GEO dataset GSE207426 (glioblastoma).

## When to use

Use when raw IDAT files plus a sample sheet from an Illumina 450K/EPIC methylation array study (e.g. from GEO) need differential methylation calling between groups, and a GUI-driven, reproducible pipeline is acceptable.

## When NOT to use

- The host is ARM64 (e.g. Apple Silicon) — methylR's Docker container is built for AMD64 and ARM64 is not supported
- Data are already normalized beta-value tables or precomputed DMC lists — use methylR's downstream accessory modules (MDS/PCA, volcano, enrichment) instead of the ChAMP differential calling module
- The analysis requires command-line scripting or cluster HPC integration beyond what the Shiny GUI offers — consider running ChAMP or minfi directly in R

## Inputs

- Raw IDAT files from an Illumina 450K or EPIC DNA methylation array (e.g. GEO accession GSE207426)
- Sample sheet (CSV) describing samples and groups
- Optionally, methylR test data (testData.zip from SourceForge) for a dry run

## Outputs

- Differential methylation probe results table (DMC list) for the full dataset
- QC, normalization, and other ChAMP-stage outputs accessible via the methylR Shiny interface

## How to apply

Download the raw IDAT files and sample sheet from GEO (e.g. GSE207426), then launch the methylR Docker container locally on an AMD64 Linux host with 'docker run --rm -p 3838:3838 jd21/methylr:latest' and open http://localhost:3838. Load the data into the methylR Shiny interface and run the ChAMP analysis module (the recommended pipeline), which executes the standard ChAMP steps: loading, quality control, normalization, and differential methylation calling. Export the differential methylation probe results table for the full dataset. Validate by checking that the total number of DMCs in the full-dataset results matches the published count (123,510 for GSE207426). Close the browser after each complete analysis to clear temporary memory before rerunning.

## Related tools

- **methylR** (Shiny-based GUI pipeline that packages ChAMP/minfi for raw methylation array loading, QC, normalization, and DMC calling) — https://github.com/JD2112/methylr
- **ChAMP** (Recommended analysis module inside methylR that performs loading, QC, normalization, and differential methylation calling)
- **Docker** (Container runtime used to launch methylR locally on AMD64 (docker run --rm -p 3838:3838 jd21/methylr:latest)) — https://github.com/JD2112/methylr-devel
- **minfi** (Alternative analysis module to ChAMP within methylR for the complete methylation analysis)

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest
```

## Evaluation signals

- Full-dataset DMC count from the exported results table equals the published 123,510 DMCs for GSE207426
- The ChAMP module completes all standard steps (loading, QC, normalization, differential methylation) without errors in the Shiny interface
- Exported DMC table contains CpG probe identifiers with differential methylation statistics for every probe
- Container runs on the supported AMD64 architecture and the Shiny app is reachable at http://localhost:3838
- Browser is closed between complete analyses so repeated runs start from cleared temporary memory (avoids stale-state artifacts)

## Limitations

- ARM64 chipset architecture is not supported by methylR; AMD64 (Intel) Linux/macOS/Windows hosts are required
- Only 450K and EPIC Illumina arrays are supported; other methylation platforms are out of scope
- After one complete analysis (ChAMP or minfi), the browser must be closed to clear temporary memory before starting again
- Docker version and browser combinations are validated only for specific tested versions (e.g. Docker 20.10.18, Firefox 105 on Ubuntu 20.04 LTS)

## Evidence

- [methods] ChAMP module is the recommended differential-calling path inside methylR: "after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again."
- [readme] methylR supports raw 450K/EPIC data analysis: [evidence span withheld — anchor status: external_doc]
- [readme] Local run via Docker on AMD64: [evidence span withheld — anchor status: external_doc]
- [methods] ARM64 unsupported: "NOTE: the ARM64 chipset architecture is actually not supported by methylR."
- [abstract] Validation by comparison with published results: "We evaluated a publicly available dataset and compared the published results with those obtained with our tool"
