---
name: dna-methylation-array-downstream-analysis
description: Use when raw or normalized Illumina 450K/EPIC methylation array data
  (e.g., GSE207426) must be analyzed for differentially methylated CpGs, or when reproducing
  published DMC counts restricted to TSS200/TSS1500 + 5'UTR annotated probes.
when_to_use_negative:
- The platform is not an Illumina 450K or EPIC DNA methylation array (e.g., WGBS or
  RNA-seq data); methylR targets array methylation specifically.
- The computing environment is ARM64 architecture, which methylR does not support
  — its Docker container is built for AMD64 only.
- Only one downstream module (e.g., pathway enrichment) is needed on an existing DMC
  table without re-running normalization or differential analysis.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: methylR
  role: Shiny-based GUI pipeline that performs normalization, differential methylation
    analysis, genomic feature annotation, and downstream modules on 450K/EPIC array
    data
  repo: https://github.com/JD2112/methylr
- name: ChAMP
  role: Selectable backend within methylR for complete differential methylation analysis
    of raw array data
- name: minfi
  role: Alternative backend within methylR for normalization/differential analysis
    and for probe annotation used to assign CpGs to TSS200/TSS1500/5'UTR features
- name: Docker
  role: Containerization used to run methylR locally (AMD64 images) for reproducible
    analyses
  repo: https://github.com/JD2112/methylr-devel
- name: R
  role: Language in which methylR and its backends are implemented; used for annotation
    joins and subsetting of DMC result tables
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:64b0693d87f88a571c3a8e511ab575316dd06251536f52d71285f25e720eab14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JD2112/methylr
  tools:
  - ChAMP
  - Docker
  - R
  - methylR
  - minfi
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# dna-methylation-array-downstream-analysis

## Summary

Runs and reproduces downstream differential methylation analyses on Illumina 450K/EPIC array data using methylR (ChAMP/minfi backends), including annotating DMCs to genomic features (TSS200, TSS1500, 5'UTR) and validating published DMC counts on public GEO datasets such as GSE207426.

## When to use

Use when raw or normalized Illumina 450K/EPIC methylation array data (e.g., GSE207426) must be analyzed for differentially methylated CpGs, or when reproducing published DMC counts restricted to TSS200/TSS1500 + 5'UTR annotated probes.

## When NOT to use

- The platform is not an Illumina 450K or EPIC DNA methylation array (e.g., WGBS or RNA-seq data); methylR targets array methylation specifically.
- The computing environment is ARM64 architecture, which methylR does not support — its Docker container is built for AMD64 only.
- Only one downstream module (e.g., pathway enrichment) is needed on an existing DMC table without re-running normalization or differential analysis.

## Inputs

- Raw Illumina 450K or EPIC DNA methylation array data (IDAT files, e.g., from GEO dataset GSE207426)
- Normalized beta-value data table (testDataFile2)
- Differentially methylated CpG results table (testDataFile1)
- Group/sample metadata (groupData)
- Probe annotation mapping CpG IDs to genomic features (TSS200, TSS1500, 5'UTR, etc.)

## Outputs

- Differentially methylated CpG (DMC) counts for the full dataset and the TSS200/TSS1500 + 5'UTR subset
- filtered_dmc_summary.csv containing the filtered subset, feature annotations used, and observed vs. reported DMC counts
- Genomic feature annotations of DMCs, volcano plots, chromosome maps, GO and pathway enrichment results

## How to apply

Run methylR locally via its Docker container (AMD64 only) or web server, choosing either ChAMP or minfi as the analysis backend for normalization and differential analysis of raw IDAT data. Load the resulting differential methylation results table, join CpG identifiers to probe annotations using minfi or ChAMP annotation facilities in R to assign each CpG a genomic feature group (TSS200, TSS1500, 5'UTR, etc.). Filter the table to CpGs annotated as TSS200, TSS1500, or 5'UTR and count them, then compare against the published reference (26,626 subset DMCs; 123,510 full-dataset DMCs for GSE207426). Write a summary report with the filtered subset, annotations used, and observed vs. reported counts. Between complete analyses, close the browser to clear temporary memory before starting another run.

## Related tools

- **methylR** (Shiny-based GUI pipeline that performs normalization, differential methylation analysis, genomic feature annotation, and downstream modules on 450K/EPIC array data) — https://github.com/JD2112/methylr
- **ChAMP** (Selectable backend within methylR for complete differential methylation analysis of raw array data)
- **minfi** (Alternative backend within methylR for normalization/differential analysis and for probe annotation used to assign CpGs to TSS200/TSS1500/5'UTR features)
- **Docker** (Containerization used to run methylR locally (AMD64 images) for reproducible analyses) — https://github.com/JD2112/methylr-devel
- **R** (Language in which methylR and its backends are implemented; used for annotation joins and subsetting of DMC result tables)

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest
```

## Evaluation signals

- Filtered TSS200/TSS1500 + 5'UTR subset DMC count equals the published reference of 26,626 DMCs for GSE207426 (output_matches_reference); any mismatch flags the run as failing.
- Full-dataset DMC count equals the published 123,510 DMCs before feature filtering.
- Every retained CpG in the subset carries a feature annotation of exactly TSS200, TSS1500, or 5'UTR in the joined probe annotation.
- filtered_dmc_summary.csv is produced and contains the filtered subset, the feature annotations used, and the observed vs. reported DMC count.
- The pipeline runs to completion in the supported environment (Linux/MacOS/Windows AMD64; ARM64 runs are invalid).

## Limitations

- ARM64 chipset architecture is not supported by methylR; the Docker container is built on AMD64 only.
- After one complete analysis with either ChAMP or minfi, the browser must be closed to clear temporary memory before starting a new run.
- MethylR was successfully tested on Mac and Linux based systems; results depend on joining CpG IDs to correct probe annotations (minfi or ChAMP).

## Evidence

- [abstract] We evaluated a publicly available dataset and compared the published results with those obtained with our tool: "We evaluated a publicly available dataset and compared the published results with those obtained with our tool"
- [methods] after one complete analysis (either with ChAMP or minfi), close the browser (to clear the temporary memory) and start again.: "after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again."
- [methods] NOTE: the ARM64 chipset architecture is actually not supported by methylR.: "NOTE: the ARM64 chipset architecture is actually not supported by *methylR*."
- [readme] a complete pipeline for the analysis of both 450K and EPIC Illumina arrays which not only offers data visualization and normalization: [evidence span withheld — anchor status: external_doc]
- [readme] docker run --rm -p 3838:3838 jd21/methylr:latest: [evidence span withheld — anchor status: external_doc]
