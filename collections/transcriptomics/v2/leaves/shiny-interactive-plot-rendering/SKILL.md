---
name: shiny-interactive-plot-rendering
description: Use when a bioinformatics pipeline needs a browser-based interactive
  figure (e.g. GO enrichment plots, volcano plots, chromosome maps) with tunable parameters,
  produced from a Shiny app served via Docker.
when_to_use_negative:
- Running on ARM64 hardware — methylR explicitly does not support the ARM64 chipset
  architecture
- The analysis output is a static publication figure rather than an interactive, parameter-selectable
  web plot
- Input is raw IDAT methylation data rather than a DMC file — the plot module expects
  downstream analysis results
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3295
tools:
- name: shiny
  role: Framework for the interactive plot with selectable parameters exposed in the
    graphical interface
  repo: https://github.com/JD2112/methylr
- name: methylR
  role: Hosts the interactive GO enrichment plot module within its Association Study
    section
  repo: https://github.com/JD2112/methylr
- name: Docker
  role: Serves the Shiny app locally via the jd21/methylr:latest container on port
    3838 (AMD64)
  repo: https://github.com/JD2112/methylr-devel
- name: R
  role: Implementation language of the Shiny plotting modules
  repo: https://github.com/JD2112/methylr
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:69107d567ae79c01f731d49c4d2f2d5c2e65de772fc4ccb3754d6bd561171a37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JD2112/methylr
  tools:
  - Docker
  - R
  - methylR
  - shiny
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# shiny-interactive-plot-rendering

## Summary

Render analysis results as interactive Shiny plots with user-selectable parameters inside a containerized Shiny Server, as done in methylR's downstream modules (e.g. the GO enrichment plot in the Association Study section) exportable as HTML.

## When to use

Use when a bioinformatics pipeline needs a browser-based interactive figure (e.g. GO enrichment plots, volcano plots, chromosome maps) with tunable parameters, produced from a Shiny app served via Docker.

## When NOT to use

- Running on ARM64 hardware — methylR explicitly does not support the ARM64 chipset architecture
- The analysis output is a static publication figure rather than an interactive, parameter-selectable web plot
- Input is raw IDAT methylation data rather than a DMC file — the plot module expects downstream analysis results

## Inputs

- DMC results file from a completed ChAMP or minfi differential methylation analysis (e.g. testDataFile1)
- Optional normalized beta value table (testDataFile2) plus groupData for PCA/MDS modules
- Selected P-value adjustment method chosen in the Shiny UI

## Outputs

- Interactive HTML plot (e.g. GO enrichment plot) rendered in the browser
- Results data table (CSV) of enrichment statistics under the chosen adjustment method

## How to apply

Wrap the plotting module as a Shiny UI so the user selects parameters interactively — in methylR's GO module, the user loads a differentially methylated CpG (DMC) file in the Association Study section and picks one of the offered P-value adjustment methods from the graphical interface. Serve the app through the published Docker container (jd21/methylr:latest, AMD64 only) with `docker run --rm -p 3838:3838` and open http://localhost:3838 in a supported browser. Run GO term overrepresentation among genes annotated to the input DMCs, then export the interactive plot as an HTML file that renders in the browser and the enrichment statistics as a results data table. Close the browser after a complete analysis to clear temporary memory before starting another run.

## Related tools

- **shiny** (Framework for the interactive plot with selectable parameters exposed in the graphical interface) — https://github.com/JD2112/methylr
- **methylR** (Hosts the interactive GO enrichment plot module within its Association Study section) — https://github.com/JD2112/methylr
- **Docker** (Serves the Shiny app locally via the jd21/methylr:latest container on port 3838 (AMD64)) — https://github.com/JD2112/methylr-devel
- **R** (Implementation language of the Shiny plotting modules) — https://github.com/JD2112/methylr

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest  # then open http://localhost:3838 and use the Association Study section to load a DMC file and export the GO plot
```

## Evaluation signals

- The exported results table lists GO terms with enrichment statistics computed under the user-selected P-value adjustment method
- The exported HTML plot renders interactively (not as a static image) in the browser
- App is reachable at http://localhost:3838 after `docker run --rm -p 3838:3838 jd21/methylr:latest`
- The chosen P-value adjustment method in the UI is reflected in the statistics of the exported table
- Browser is closed between analyses so temporary memory is cleared before the next module run

## Limitations

- ARM64 chipset architecture is not supported; the Docker container must run on AMD64 (Linux, Intel macOS, or Windows with WSL2)
- Browser compatibility varies by OS: e.g. Firefox 105 on Ubuntu 20.04LTS, Chrome 106/Safari 15.6.1 on macOS Monterey, Chrome/Edge/Firefox 106–107 on Windows 10 21H2
- Browser must be closed after a complete analysis to clear temporary memory before starting another analysis

## Evidence

- [abstract] Interactive GO plot with adjustable parameters: [evidence span withheld — anchor status: not_found]
- [readme] Export validation: [evidence span withheld — anchor status: not_found]
- [readme] Docker serving command: [evidence span withheld — anchor status: external_doc]
- [methods] Architecture limitation: "NOTE: the ARM64 chipset architecture is actually not supported by *methylR*."
