---
name: go-enrichment-analysis-of-dmc-annotated-genes
description: Use when you have a DMC results file from a methylR differential methylation
  run (ChAMP or minfi) and need functional interpretation via GO term enrichment of
  the genes annotated to those CpGs.
when_to_use_negative:
- Your input is raw IDAT or normalized beta values rather than a DMC results file
  — the GO module requires output of a completed ChAMP or minfi differential analysis
- You need only exploratory plots (MDS/PCA/volcano) that consume beta-value matrices
  (testDataFile2), not DMC-derived gene lists
- You are running on ARM64 chipset architecture, which methylR does not support
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: methylR
  role: Shiny-based graphical application whose Association Study section hosts the
    GO enrichment module over genes annotated to input DMCs
  repo: https://github.com/JD2112/methylr
- name: ChAMP
  role: Upstream differential methylation engine that generates the DMC file consumed
    by the GO module
- name: minfi
  role: Alternative upstream differential methylation engine generating the DMC file
    for the GO module
- name: Docker
  role: Container runtime used to launch the methylR Shiny app locally on AMD64 systems
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:8f0bca726689520d628a2eba251dbe7a2e2147b70ce4b9a647a2e2f978637b63
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# go-enrichment-analysis-of-dmc-annotated-genes

## Summary

Runs the Gene Ontology enrichment module of methylR, which tests GO term overrepresentation among genes annotated to differentially methylated CpGs (DMCs) from a completed ChAMP or minfi analysis, and renders results as an interactive plot plus a downloadable data table.

## When to use

Use when you have a DMC results file from a methylR differential methylation run (ChAMP or minfi) and need functional interpretation via GO term enrichment of the genes annotated to those CpGs.

## When NOT to use

- Your input is raw IDAT or normalized beta values rather than a DMC results file — the GO module requires output of a completed ChAMP or minfi differential analysis
- You need only exploratory plots (MDS/PCA/volcano) that consume beta-value matrices (testDataFile2), not DMC-derived gene lists
- You are running on ARM64 chipset architecture, which methylR does not support

## Inputs

- DMC results file from a completed ChAMP or minfi differential methylation analysis in methylR
- methylR testDataFile1 (differentially methylated data file from testData.zip at SourceForge)

## Outputs

- Interactive GO enrichment plot exported as an HTML file
- GO enrichment results data table (CSV) listing GO terms with enrichment statistics under the chosen P-value adjustment method

## How to apply

Obtain the DMC results file — either from a completed ChAMP or minfi analysis inside methylR or from testDataFile1 in the methylR test data, which is explicitly listed as usable for the gene ontology module. Launch methylR in its Docker container (AMD64 only) and open the Shiny interface at http://localhost:3838; navigate to the Association Study section and load the DMC file. Select one of the offered P-value adjustment methods from the graphical interface, then execute the GO enrichment module, which tests Gene Ontology term overrepresentation among the genes annotated to the input differentially methylated CpGs. Export the interactive enrichment plot as an HTML file and the enrichment statistics as a results data table (CSV). Because Shiny holds state in temporary memory, close the browser after a complete analysis before starting another one.

## Related tools

- **methylR** (Shiny-based graphical application whose Association Study section hosts the GO enrichment module over genes annotated to input DMCs) — https://github.com/JD2112/methylr
- **ChAMP** (Upstream differential methylation engine that generates the DMC file consumed by the GO module)
- **minfi** (Alternative upstream differential methylation engine generating the DMC file for the GO module)
- **Docker** (Container runtime used to launch the methylR Shiny app locally on AMD64 systems)

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest  # then open http://localhost:3838, load the DMC file in the Association Study section, choose a P-value adjustment, and run the GO module
```

## Evaluation signals

- The exported CSV table lists GO terms with enrichment statistics computed under the P-value adjustment method selected in the UI
- The exported HTML plot renders interactively in a supported web browser (e.g., Firefox 105+, Chrome 106+, Safari 15.6.1)
- The input DMC file was accepted in the Association Study section without errors (valid DMC results format from ChAMP or minfi)
- Enriched GO terms correspond to genes annotated to the input DMCs, not the full CpG array background
- The browser was closed after the analysis to clear Shiny temporary memory before a subsequent run

## Limitations

- methylR does not support ARM64 chipset architecture; the Docker container must run on AMD64 (Linux, macOS Intel, or Windows with WSL2)
- Shiny holds analysis state in temporary memory, so the browser must be closed after one complete analysis before starting another
- The GO module depends on the upstream ChAMP or minfi differential analysis being completed first; it cannot run on raw methylation files directly

## Evidence

- [readme] testDataFile1 usable for GO module: [evidence span withheld — anchor status: not_found]
- [abstract] eight downstream modules to pathway enrichment: "We implemented eight downstream analysis modules that can perform multidimensional analyses to pathway enrichment"
- [methods] close browser to clear memory: "after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again."
- [readme] Docker container run command: [evidence span withheld — anchor status: external_doc]
- [readme] ARM64 not supported: "PLEASE NOTE: the ARM64 chipset architecture is actually not supported by *methylR*."
