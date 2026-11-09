---
name: mirna-expression-arm-selection-boxplotting
description: Use when manually curating candidate plant miRNAs and needing to inspect
  arm-specific expression and 5p/3p arm-switching events of a selected precursor across
  available sRNA-Seq samples, as part of the manualCuration HTML report.
when_to_use_negative:
- The sRNA-Seq data come from a species without a reference genome — iwa-miRNA ignores
  reads that do not map to the reference genome and cannot handle such data.
- The goal is downstream enrichment or cross-species comparative analysis of curated
  miRNAs — the article notes iwa-miRNA has limited ability for such downstream analysis.
- The candidate precursor has no supporting sRNA-Seq samples, so no arm expression/arm-selection
  events can be visualized.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
tools:
- name: iwa-miRNA
  role: Provides the manualCuration module that generates the per-miRNA report page
    containing the arm-expression boxplot and arm-selection events
  repo: https://github.com/cma2015/iwa-miRNA
- name: Rmarkdown
  role: Renders the boxplot and other feature views into the interactive HTML report
    page
- name: Galaxy
  role: Framework within which the manualCuration report generation is organized
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:b01674c515b190b0d67fd34a7362726fcf1400cbdd03f5d3dbcd7758dfc32b2d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Galaxy
  - Rmarkdown
  - iwa-miRNA
  - psRNAtarget
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mirna-expression-arm-selection-boxplotting

## Summary

Generates the expression-pattern panel of an iwa-miRNA manualCuration report: a boxplot of miRNA arm expression across sRNA-Seq samples that visualizes miRNA arm selection (5p vs 3p) events, rendered via Rmarkdown into the per-miRNA interactive HTML page.

## When to use

Use when manually curating candidate plant miRNAs and needing to inspect arm-specific expression and 5p/3p arm-switching events of a selected precursor across available sRNA-Seq samples, as part of the manualCuration HTML report.

## When NOT to use

- The sRNA-Seq data come from a species without a reference genome — iwa-miRNA ignores reads that do not map to the reference genome and cannot handle such data.
- The goal is downstream enrichment or cross-species comparative analysis of curated miRNAs — the article notes iwa-miRNA has limited ability for such downstream analysis.
- The candidate precursor has no supporting sRNA-Seq samples, so no arm expression/arm-selection events can be visualized.

## Inputs

- Selected miRNA candidate records from the MiRNA Selection module (e.g., ath-MIR156b)
- Read counts / expression values of mature miRNA arms (5p and 3p) across available sRNA-Seq samples
- Test dataset from https://github.com/cma2015/iwa-miRNA/tree/master/Test_data

## Outputs

- Boxplot of miRNA expression patterns and arm-selection events across sRNA-Seq samples, embedded in the per-miRNA manualCuration HTML report page (e.g., ath-MIR156b_report.html)

## How to apply

Load the selected miRNA candidate record (e.g., ath-MIR156b) from the manualCuration inputs, using test data from the iwa-miRNA GitHub Test_data directory. For the chosen precursor, aggregate mature-miRNA read counts from the available sRNA-Seq samples and plot a boxplot of expression patterns that also marks arm-selection events (which arm, miR-5p or miR-3p, dominates in each sample). Render this panel alongside the secondary structure and read-stack views in the per-miRNA HTML report page produced with Rmarkdown within the Galaxy framework, so curators can judge expression support and arm usage interactively. Correctness is judged by the boxplot appearing in the report with arm-selection events visible across samples.

## Related tools

- **iwa-miRNA** (Provides the manualCuration module that generates the per-miRNA report page containing the arm-expression boxplot and arm-selection events) — https://github.com/cma2015/iwa-miRNA
- **Rmarkdown** (Renders the boxplot and other feature views into the interactive HTML report page)
- **Galaxy** (Framework within which the manualCuration report generation is organized)

## Examples

```
Rscript -e "rmarkdown::render('ath-MIR156b_report.Rmd', output_file='ath-MIR156b_report.html')"
```

## Evaluation signals

- The per-miRNA report HTML file (e.g., ath-MIR156b_report.html) exists and is a well-formed HTML document
- The report page contains an expression boxplot visualizing the miRNA's expression patterns across the available sRNA-Seq samples
- The boxplot displays arm-selection events (5p vs 3p dominance) across different samples
- The boxplot is rendered in the Rmarkdown-based HTML page within the Galaxy framework alongside the other required feature views (secondary structure, read stacks, target network)

## Limitations

- Reads that do not map to the reference genome are ignored in the current version of iwa-miRNA, so unmapped arm reads are not reflected in the boxplot.
- Aggregated candidate lists can contain false positives; wet-lab validation is still required before functional interpretation of curated candidates.
- Limited downstream analysis of selected miRNAs (e.g., no GO annotation or miRNA set enrichment analysis) is available beyond the report visualizations.

## Evidence

- [methods] the iwa-miRNA report page generated for each selected miRNA includes read support of miRNA, miRNA arm expression patterns, arm selection events across different samples: "read support of miRNA, miRNA arm expression patterns, arm selection events across different samples"
- [methods] Rmarkdown renders the report: "summarizes the results into HTML pages, using Rmarkdown for easy visualization, interpretation, and sharing"
- [methods] manualCuration module produces report pages: [evidence span withheld — anchor status: not_found]
- [readme] Readme test data pointer: "Test datasets referred in the tutorials for iwa-miRNA: https://github.com/cma2015/iwa-miRNA/tree/master/Test_data"
