---
name: multiple-testing-p-adjustment-selection
description: Use when running methylR's Association Study downstream modules such
  as GO or pathway enrichment on a DMC results file and an explicit multiple-testing
  correction choice must be made through the graphical interface before statistics
  and plots are generated.
when_to_use_negative:
- Input is raw IDAT or normalized beta-value data rather than a DMC results file —
  GO enrichment requires the differential methylation output, while beta tables are
  designated for MDS/PCA plots
- The task is differential methylation calling itself (ChAMP or minfi step) rather
  than post-hoc enrichment, where the adjustment applies to enrichment statistics,
  not per-CpG raw P-values
- Running on ARM64 hardware, which methylR does not support
edam_operation: http://edamontology.org/operation_0305
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3173
tools:
- name: methylR
  role: Shiny-based graphical interface that exposes selectable P-value adjustment
    methods and runs the GO enrichment module on the loaded DMC file
  repo: https://github.com/JD2112/methylr
- name: ChAMP
  role: One of the upstream pipelines whose DMC results file serves as input to the
    enrichment module
- name: minfi
  role: Alternative upstream pipeline producing the DMC results file consumed by the
    enrichment module
- name: Docker
  role: Container runtime used to launch methylR locally (docker run --rm -p 3838:3838
    jd21/methylr:latest) on AMD64 systems
  repo: https://github.com/JD2112/methylr-devel
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:75c16206618cec395f6c3321a7e9dbdf33fddfec30193f61b7aa185699117b82
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
  edam_operation: http://edamontology.org/operation_0305
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# multiple-testing-p-adjustment-selection

## Summary

Select and apply one of the P-value adjustment methods offered by methylR's graphical interface when running downstream modules (e.g., the Gene Ontology enrichment analysis), so that enrichment statistics reported for differentially methylated CpGs are corrected for multiple testing under a consistent, user-chosen method.

## When to use

Use when running methylR's Association Study downstream modules such as GO or pathway enrichment on a DMC results file and an explicit multiple-testing correction choice must be made through the graphical interface before statistics and plots are generated.

## When NOT to use

- Input is raw IDAT or normalized beta-value data rather than a DMC results file — GO enrichment requires the differential methylation output, while beta tables are designated for MDS/PCA plots
- The task is differential methylation calling itself (ChAMP or minfi step) rather than post-hoc enrichment, where the adjustment applies to enrichment statistics, not per-CpG raw P-values
- Running on ARM64 hardware, which methylR does not support

## Inputs

- DMC results file from a completed ChAMP or minfi differential methylation analysis
- methylR testDataFile1 (differentially methylated data file) from github.com/JD2112/methylr/data
- Selected P-value adjustment method chosen in the methylR Shiny interface

## Outputs

- Results data table (CSV) of GO terms with multiple-testing-adjusted enrichment statistics
- Interactive GO enrichment plot exported as an HTML file

## How to apply

Obtain the DMC results file produced by a completed ChAMP or minfi differential methylation analysis (or the testDataFile1 from the methylr GitHub data directory, which is designated for gene ontology and pathway enrichment modules). Launch the methylR Shiny app in its Docker container (AMD64 only) and, in the Association Study section, load the DMC file. Use the graphical interface to select one of the offered P-value adjustment methods; the GO module then tests Gene Ontology term overrepresentation among genes annotated to the input differentially methylated CpGs, applying the chosen correction to the enrichment statistics. Rationale: the article emphasizes that methylR 'offers ... a wide range of P-value adjustment methods' exposed interactively, so the adjustment is a user-facing decision point rather than a fixed default. Export the interactive GO plot as HTML and the enrichment statistics as a results table (CSV), then validate that the exported table lists GO terms with enrichment statistics under the chosen adjustment method.

## Related tools

- **methylR** (Shiny-based graphical interface that exposes selectable P-value adjustment methods and runs the GO enrichment module on the loaded DMC file) — https://github.com/JD2112/methylr
- **ChAMP** (One of the upstream pipelines whose DMC results file serves as input to the enrichment module)
- **minfi** (Alternative upstream pipeline producing the DMC results file consumed by the enrichment module)
- **Docker** (Container runtime used to launch methylR locally (docker run --rm -p 3838:3838 jd21/methylr:latest) on AMD64 systems) — https://github.com/JD2112/methylr-devel

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest  # then open http://localhost:3838, load the DMC file in the Association Study section, and select a P-value adjustment method
```

## Evaluation signals

- The exported results table lists GO terms with enrichment statistics computed under the specific P-value adjustment method selected in the interface
- The exported HTML GO plot renders interactively in the browser
- The enrichment gene universe corresponds to genes annotated to the input differentially methylated CpGs from the loaded DMC file
- A completed upstream ChAMP or minfi analysis exists before enrichment is run, and the browser was closed between analyses to clear temporary memory

## Limitations

- ARM64 chipset architecture is not supported by methylR; the Docker container is built on AMD64 OS architecture only
- After one complete ChAMP or minfi analysis, the browser must be closed to clear temporary memory before starting again
- Only the P-value adjustment methods offered in the methylR graphical interface are available; no programmatic override is described

## Evidence

- [other] The GO analysis module in methylR offers an interactive plot with selectable parameters and a wide range of P-value adjustment methods, and it generates a results data table for the user.: [evidence span withheld — anchor status: not_found]
- [readme] methylR is packed into a Docker container available online and was tested on AMD64 systems only.: [evidence span withheld — anchor status: external_doc]
- [methods] after one complete analysis (either with ChAMP or minfi), close the browser (to clear the temporary memory) and start again.: "after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again."
- [methods] the ARM64 chipset architecture is actually not supported by methylR.: "NOTE: the ARM64 chipset architecture is actually not supported by *methylR*."
