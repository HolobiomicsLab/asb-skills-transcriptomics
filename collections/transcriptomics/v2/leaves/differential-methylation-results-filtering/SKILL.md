---
name: differential-methylation-results-filtering
description: Use when you have a DMC results table from Illumina 450K/EPIC methylation
  analysis and need to restrict CpGs to promoter/5'UTR features or apply adjusted-P/logFC
  thresholds before visualization or enrichment.
when_to_use_negative:
- Input is already a normalized beta value matrix (testDataFile2) intended for MDS/PCA
  plots rather than DMC filtering
- The differential methylation analysis itself has not been run — no DMC results file
  exists yet, so ChAMP/minfi differential analysis must be completed first
- Working on ARM64 chipset architecture, which methylR does not support
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: methylR
  role: Shiny-based GUI hosting the Feature Analysis section where adjusted-P/logFC
    cutoffs are applied and the filtered CpG list and volcano plot are generated
  repo: https://github.com/JD2112/methylr
- name: minfi
  role: Alternative backend engine within methylR that produces the DMC results and
    provides probe annotation used to assign feature groups
- name: ChAMP
  role: Alternative backend engine within methylR producing the differential methylation
    table that is then filtered and annotated
- name: Docker
  role: Container runtime for launching methylR locally on AMD64 systems to run the
    filtering workflow
  repo: https://github.com/JD2112/methylr-devel/actions/workflows/docker-image.yml
- name: R
  role: Underlying language for the minfi/ChAMP annotation join and subset counting
    steps
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:b7dfc4b04d19de0c1c2d72a51e1bcd0f0f8baba6e6bdae1aaec707450a785c2d
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# differential-methylation-results-filtering

## Summary

Filters and post-processes differentially methylated CpG (DMC) result tables produced by ChAMP- or minfi-based differential analysis in methylR, subsetting probes by genomic feature annotation (TSS200, TSS1500, 5'UTR) and by adjusted P-value and logFC cutoffs for downstream feature-analysis modules.

## When to use

Use when you have a DMC results table from Illumina 450K/EPIC methylation analysis and need to restrict CpGs to promoter/5'UTR features or apply adjusted-P/logFC thresholds before visualization or enrichment.

## When NOT to use

- Input is already a normalized beta value matrix (testDataFile2) intended for MDS/PCA plots rather than DMC filtering
- The differential methylation analysis itself has not been run — no DMC results file exists yet, so ChAMP/minfi differential analysis must be completed first
- Working on ARM64 chipset architecture, which methylR does not support

## Inputs

- DMC results table (testDataFile1) from ChAMP- or minfi-based differential methylation analysis in methylR
- CpG probe annotation mapping (genomic feature groups: TSS200, TSS1500, 5'UTR, etc.) from minfi or ChAMP
- User-selected adjusted P-value and logFC cutoffs
- Normalized beta value matrix (testDataFile2) with groupData if contextual sample groups are needed

## Outputs

- Filtered subset of CpGs restricted to TSS200/TSS1500/5'UTR annotations with feature labels
- List of significant CpG IDs passing both adjusted-P and logFC cutoffs, alongside an interactive volcano plot
- Summary CSV (filtered_dmc_summary.csv) with observed vs. reported DMC counts

## How to apply

Load the DMC results file (e.g., testDataFile1 from the methylR testData.zip deposit, produced by a completed ChAMP or minfi differential analysis inside the methylR Shiny app). Join CpG identifiers to probe annotations using minfi or ChAMP annotation facilities to assign each CpG a genomic feature group (TSS200, TSS1500, 5'UTR, etc.), then retain only CpGs annotated to the features of interest. For volcano-style filtering, apply the user-selected adjusted P-value cutoff first, then the log-fold-change cutoff, rendering -log10(adjusted P) versus logFC and exporting only CpGs passing both thresholds. For published-result reproduction, count the filtered subset and compare it against the reference value — e.g., methylR reproduced GSE207426's 123,510 total DMCs and 26,626 DMCs in the TSS200/TSS1500 + 5'UTR subset. Write the filtered subset, feature annotations used, and observed-vs-reported counts to a summary CSV (e.g., filtered_dmc_summary.csv). Close the browser after each complete analysis to clear temporary memory before rerunning with different cutoffs.

## Related tools

- **methylR** (Shiny-based GUI hosting the Feature Analysis section where adjusted-P/logFC cutoffs are applied and the filtered CpG list and volcano plot are generated) — https://github.com/JD2112/methylr
- **minfi** (Alternative backend engine within methylR that produces the DMC results and provides probe annotation used to assign feature groups)
- **ChAMP** (Alternative backend engine within methylR producing the differential methylation table that is then filtered and annotated)
- **Docker** (Container runtime for launching methylR locally on AMD64 systems to run the filtering workflow) — https://github.com/JD2112/methylr-devel/actions/workflows/docker-image.yml
- **R** (Underlying language for the minfi/ChAMP annotation join and subset counting steps)

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest
```

## Evaluation signals

- The filtered TSS200/TSS1500/5'UTR subset count matches the published reference value of 26,626 DMCs (output_matches_reference); any mismatch flags the run as failing
- The full-dataset DMC count matches the published 123,510 DMCs before subsetting
- Every CpG in the exported significant list lies beyond both the adjusted-P and logFC cutoffs displayed on the volcano plot (plot/list mutual consistency)
- Each retained CpG carries a valid genomic feature annotation drawn from the minfi/ChAMP probe mapping
- The summary CSV contains the filtered subset, the feature annotations used, and observed vs. reported DMC counts

## Limitations

- methylR is not supported on ARM64 chipset architecture; the Docker container is built for AMD64 (Linux, Intel macOS, Windows with WSL2)
- The web server moved from methylr.research.liu.se to methylr.serve.scilifelab.se — older URLs may fail
- The browser must be closed after each complete ChAMP/minfi analysis to clear temporary memory before starting another run
- Filtering depends on the DMC results file existing from a completed upstream differential analysis; it cannot be run on raw IDAT data directly

## Evidence

- [readme] testDataFile1 is usable for all feature-analysis filtering modules: [evidence span withheld — anchor status: not_found]
- [methods] Browser must be closed after each ChAMP/minfi run: "after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again."
- [readme] methylR is a complete 450K/EPIC pipeline including annotation of genomic features from the analysis: [evidence span withheld — anchor status: external_doc]
- [abstract] Eight downstream modules span multidimensional analysis to pathway enrichment: "We implemented eight downstream analysis modules that can perform multidimensional analyses to pathway enrichment"
- [methods] ARM64 architecture unsupported: "NOTE: the ARM64 chipset architecture is actually not supported by methylR."
