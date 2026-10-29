---
name: rmarkdown-html-report-rendering
description: Use when a selected miRNA candidate (e.g., ath-MIR156b from manualCuration
  output) must be presented as a feature-rich, shareable HTML report page for manual
  curation in a Galaxy-based plant miRNA annotation workflow.
when_to_use_negative:
- The goal is downstream enrichment or comparative analysis of selected miRNAs (e.g.,
  GO annotation, miRNA set enrichment, cross-species comparison) — iwa-miRNA's report
  pages do not provide these.
- The sRNA-Seq data come from a species without a reference genome, or the candidate
  derives from unmapped reads — the report pipeline relies on genome-mapped candidates.
- A machine-readable candidate table is already sufficient and no human-facing curation
  report is needed.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0092
tools:
- name: iwa-miRNA
  role: Galaxy-based framework whose manualCuration module generates the summary and
    per-miRNA report pages
  repo: https://github.com/cma2015/iwa-miRNA
- name: Rmarkdown
  role: Rendering engine that summarizes manual curation results into interactive
    HTML report pages for visualization, interpretation, and sharing
- name: psRNAtarget
  role: Predicts miRNA–target interactions depicted in the bipartite network on each
    report page
- name: Galaxy
  role: Framework organizing the modules and dependencies, including the Rmarkdown
    report rendering step
  repo: https://github.com/cma2015/iwa-miRNA
- name: Docker
  role: Docker image packaging iwa-miRNA so the report-generation environment can
    be reproduced locally
  repo: https://hub.docker.com/r/malab/iwa-mirna
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:8d0d6854b0753373a869fcbe720ed737b488bceb657c9d69e337e4d8bafd1f4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Docker
  - Galaxy
  - Rmarkdown
  - iwa-miRNA
  - psRNAtarget
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# rmarkdown-html-report-rendering

## Summary

Render per-miRNA manual curation report pages as interactive HTML documents using Rmarkdown within the iwa-miRNA Galaxy framework, summarizing secondary structure, read support, expression/arm-selection, and target-interaction views for each selected miRNA candidate.

## When to use

Use when a selected miRNA candidate (e.g., ath-MIR156b from manualCuration output) must be presented as a feature-rich, shareable HTML report page for manual curation in a Galaxy-based plant miRNA annotation workflow.

## When NOT to use

- The goal is downstream enrichment or comparative analysis of selected miRNAs (e.g., GO annotation, miRNA set enrichment, cross-species comparison) — iwa-miRNA's report pages do not provide these.
- The sRNA-Seq data come from a species without a reference genome, or the candidate derives from unmapped reads — the report pipeline relies on genome-mapped candidates.
- A machine-readable candidate table is already sufficient and no human-facing curation report is needed.

## Inputs

- Selected miRNA candidate records from the MiRNA Selection module (manualCuration input)
- Test datasets from https://github.com/cma2015/iwa-miRNA/tree/master/Test_data (e.g., ath-MIR156b record)
- sRNA-Seq-derived read support and expression/arm-selection data for the candidate miRNA
- miRNA–target interactions predicted by psRNAtarget

## Outputs

- Per-miRNA interactive HTML report page rendered via Rmarkdown (e.g., ath-MIR156b_report.html) containing secondary structure plot, read stacks, expression boxplot, and target-interaction network
- Summary and report pages from the manualCuration module

## How to apply

Load the selected miRNA candidate record from the manualCuration inputs (test data supplied in the iwa-miRNA GitHub Test_data directory), then generate each feature view in its respective plotting style: an RNA secondary structure plot locating mature miRNA arms (e.g., miR156b-5p and miR156b-3p) within the precursor together with quality-profiling results; read stacks showing read support of the identified miRNA; a boxplot of expression patterns and arm-selection events across available sRNA-Seq samples; and a bipartite network of miRNA–target interactions predicted by psRNAtarget. Render all feature views into an interactive HTML page with Rmarkdown, organized within the Galaxy framework, so results are easy to visualize, interpret, and share. Validate the output by checking that the per-miRNA report HTML exists, is well-formed, and contains all required feature fields.

## Related tools

- **iwa-miRNA** (Galaxy-based framework whose manualCuration module generates the summary and per-miRNA report pages) — https://github.com/cma2015/iwa-miRNA
- **Rmarkdown** (Rendering engine that summarizes manual curation results into interactive HTML report pages for visualization, interpretation, and sharing)
- **psRNAtarget** (Predicts miRNA–target interactions depicted in the bipartite network on each report page)
- **Galaxy** (Framework organizing the modules and dependencies, including the Rmarkdown report rendering step) — https://github.com/cma2015/iwa-miRNA
- **Docker** (Docker image packaging iwa-miRNA so the report-generation environment can be reproduced locally) — https://hub.docker.com/r/malab/iwa-mirna

## Evaluation signals

- The per-miRNA report file (e.g., ath-MIR156b_report.html) exists at the expected output location.
- The report is a well-formed HTML document (parses without structural errors).
- The report contains the required feature fields: RNA secondary structure plot with mature miRNA arm locations, read stacks of miRNA read support, expression/arm-selection boxplot, and miRNA–target interaction bipartite network.
- The rendered page matches the manualCuration module's documented output (summary and report pages) as described in the tutorial and Test_data examples.
- Feature views are rendered in their respective styles customized by feature type, consistent with the article's description.

## Limitations

- iwa-miRNA ignores sequencing reads that do not map to the reference genome, so report content reflects only genome-mapped evidence.
- It cannot handle sRNA-Seq data from species without genome sequences, restricting report generation to species with reference genomes.
- Downstream analysis in the reports is limited — no enrichment analyses (e.g., GO annotation or miRNA set enrichment) or cross-species comparative analysis are included.
- Reported candidates may contain false positives; wet-lab validation is still needed before functional investigation.

## Evidence

- [methods] summarizes the results into HTML pages, using Rmarkdown for easy visualization, interpretation, and sharing.: "summarizes the results into HTML pages, using Rmarkdown for easy visualization, interpretation, and sharing."
- [methods] A bipartite network is constructed to depict miRNA–target interactions predicted by psRNAtarget.: "A bipartite network is constructed to depict miRNA–target interactions predicted by psRNAtarget [36]."
- [methods] The iwa-miRNA comprises three modules: MiRNA Compilation, MiRNA Selection, and Manual Curation: "The iwa-miRNA comprises three modules: MiRNA Compilation, MiRNA Selection, and Manual Curation (Figure 1; Table 1)."
- [readme] Test datasets referred in the tutorials for iwa-miRNA: "Test datasets referred in the tutorials for iwa-miRNA: https://github.com/cma2015/iwa-miRNA/tree/master/Test_data"
- [methods] The source codes of the modules and their dependencies are fully organized within the Galaxy framework.: "The source codes of the modules and their dependencies are fully organized within the Galaxy framework."
