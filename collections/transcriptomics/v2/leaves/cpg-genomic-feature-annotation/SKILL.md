---
name: cpg-genomic-feature-annotation
description: Use when you have a table of differentially methylated CpGs from a 450K/EPIC
  Illumina array analysis and need to join CpG IDs to probe annotations, subset by
  genomic feature (TSS/5'UTR), or validate a published feature-restricted DMC count.
when_to_use_negative:
- Input is raw IDAT data that has not yet been processed into a differential methylation
  results table (run the methylR/ChAMP/minfi pipeline first)
- Analysis targets whole-genome sequencing or ENzyme/cpg-free methylation data rather
  than Illumina 450K/EPIC array CpG probes
- The published reference used different genomic feature categories or thresholds,
  making count comparison invalid
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: methylR
  role: Shiny graphical environment that provides genomic feature annotation of results
    from 450K/EPIC array differential methylation analysis
  repo: https://github.com/JD2112/methylr
- name: minfi
  role: Used within methylR to perform the differential analysis and provide probe
    annotations for assigning CpG feature groups
- name: ChAMP
  role: Alternative pipeline within methylR for differential analysis and CpG feature
    annotation
- name: Docker
  role: Container to run methylR locally (AMD64 architecture) for the annotation workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:0f9c4eeb6b74affc3bffe5cdc5133052428b4207d98608345811f01ed3f53866
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cpg-genomic-feature-annotation

## Summary

Annotate CpG identifiers from differential methylation results with their genomic feature groups (e.g., TSS200, TSS1500, 5'UTR) using minfi or ChAMP annotation facilities, then filter and count the annotated subset to reproduce published DMC counts.

## When to use

Use when you have a table of differentially methylated CpGs from a 450K/EPIC Illumina array analysis and need to join CpG IDs to probe annotations, subset by genomic feature (TSS/5'UTR), or validate a published feature-restricted DMC count.

## When NOT to use

- Input is raw IDAT data that has not yet been processed into a differential methylation results table (run the methylR/ChAMP/minfi pipeline first)
- Analysis targets whole-genome sequencing or ENzyme/cpg-free methylation data rather than Illumina 450K/EPIC array CpG probes
- The published reference used different genomic feature categories or thresholds, making count comparison invalid

## Inputs

- Differential methylation results table (CpG IDs with statistics) from ChAMP/minfi or methylR analysis
- CpG probe annotation objects from minfi or ChAMP (450K or EPIC Illumina arrays)
- Reference DMC count from a published study (e.g., 26,626 DMCs in the TSS200/TSS1500 + 5'UTR subset)

## Outputs

- Filtered subset of DMCs annotated as TSS200, TSS1500, or 5'UTR
- Summary report (filtered_dmc_summary.csv) with feature annotations and observed vs. reported DMC counts

## How to apply

Load the differential methylation results table (e.g., for GSE207426 produced by methylR/ChAMP or minfi) and join the CpG identifiers to their probe annotations using the annotation facilities of minfi or ChAMP within R, assigning each CpG its genomic feature group (TSS200, TSS1500, 5'UTR, etc.). Subset the results table, retaining only CpGs whose feature annotation is TSS200, TSS1500, or 5'UTR, then count the CpGs in the filtered subset and compare against the published reference value (26,626 DMCs for the GSE207426 TSS200/TSS1500 + 5'UTR subset; 123,510 for the full set). Write a summary report containing the filtered subset, the feature annotations used, and the observed vs. reported DMC count. Run the analysis within methylR's Shiny/Docker environment, closing the browser after each complete ChAMP or minfi analysis to clear temporary memory before starting again.

## Related tools

- **methylR** (Shiny graphical environment that provides genomic feature annotation of results from 450K/EPIC array differential methylation analysis) — https://github.com/JD2112/methylr
- **minfi** (Used within methylR to perform the differential analysis and provide probe annotations for assigning CpG feature groups)
- **ChAMP** (Alternative pipeline within methylR for differential analysis and CpG feature annotation)
- **Docker** (Container to run methylR locally (AMD64 architecture) for the annotation workflow)

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest  # open http://localhost:3838, run minfi/ChAMP analysis, then annotate CpGs and subset to TSS200/TSS1500/5'UTR in R
```

## Evaluation signals

- The filtered TSS200/TSS1500/5'UTR subset count equals 26,626 DMCs, matching the published GSE207426 value
- The full unfiltered DMC count equals 123,510, matching the first published validation figure
- Every CpG in the output carries a non-empty feature annotation from the minfi/ChAMP annotation join
- filtered_dmc_summary.csv contains the subset, the annotations used, and both observed and reported counts
- No CpGs are dropped during the annotation join (row counts before and after annotation match)

## Limitations

- methylR does not support ARM64 chipset architecture; the Docker container is built for AMD64 only (Ubuntu 20.04 LTS, Docker 20.10.18 tested on Linux; Intel Mac and Windows 10 WSL2 supported)
- After one complete ChAMP or minfi analysis the browser must be closed to clear temporary memory before starting another run
- Annotation categories and therefore subset counts depend on which feature groups (TSS200, TSS1500, 5'UTR, etc.) are selected; a mismatch with a published count flags the run as failing the comparison

## Evidence

- [methods] Join CpG identifiers to probe annotations to assign feature groups: [evidence span withheld — anchor status: not_found]
- [methods] Subset to TSS/5'UTR features and count: [evidence span withheld — anchor status: not_found]
- [readme] methylR provides genomic feature annotation and DMC comparisons: [evidence span withheld — anchor status: external_doc]
- [methods] Close browser after complete analysis: "after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again."
- [readme] ARM64 not supported: "**PLEASE NOTE: the ARM64 chipset architecture is actually not supported by *methylR*.**"
