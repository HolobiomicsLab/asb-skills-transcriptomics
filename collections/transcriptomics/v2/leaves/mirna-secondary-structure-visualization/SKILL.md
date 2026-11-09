---
name: mirna-secondary-structure-visualization
description: Use when reconstructing or inspecting the manualCuration report generator
  of iwa-miRNA, or when visualizing a selected plant miRNA precursor's secondary structure,
  read support, arm expression, and targets in an HTML page.
when_to_use_negative:
- The species has no reference genome, since iwa-miRNA cannot handle sRNA-Seq data
  from species without genome sequences
- The goal is downstream enrichment analysis (e.g., GO annotation or miRNA set enrichment)
  or cross-species comparative analysis, which iwa-miRNA's exploratory analysis does
  not support
- Input reads do not map to the reference genome, as the current version ignores unmapped
  sequencing reads
edam_operation: http://edamontology.org/operation_0278
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
tools:
- name: iwa-miRNA
  role: Galaxy-based framework whose manualCuration module produces summary and report
    pages for selected miRNA candidates
  repo: https://github.com/cma2015/iwa-miRNA
- name: Rmarkdown
  role: Renders the per-miRNA feature views into HTML report pages for easy visualization,
    interpretation, and sharing
- name: psRNAtarget
  role: Predicts miRNA–target interactions that are depicted as a bipartite network
    in the report page
- name: Galaxy
  role: Hosts the manualCuration module and its report-generation workflow within
    a user-friendly web interface
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:bf4987a05cc9c1746dae6708d78be46d48835d003b893a0ad0f5224a942aff79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Galaxy
  - Rmarkdown
  - iwa-miRNA
  - psRNAtarget
  edam_operation: http://edamontology.org/operation_0278
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mirna-secondary-structure-visualization

## Summary

Generates per-miRNA HTML report pages within the iwa-miRNA manualCuration module, rendering RNA secondary structure with mature miRNA arm locations, read-support stacks, expression/arm-selection boxplots, and psRNAtarget-derived miRNA–target interaction networks using Rmarkdown.

## When to use

Use when reconstructing or inspecting the manualCuration report generator of iwa-miRNA, or when visualizing a selected plant miRNA precursor's secondary structure, read support, arm expression, and targets in an HTML page.

## When NOT to use

- The species has no reference genome, since iwa-miRNA cannot handle sRNA-Seq data from species without genome sequences
- The goal is downstream enrichment analysis (e.g., GO annotation or miRNA set enrichment) or cross-species comparative analysis, which iwa-miRNA's exploratory analysis does not support
- Input reads do not map to the reference genome, as the current version ignores unmapped sequencing reads

## Inputs

- Selected miRNA candidate records from the MiRNA Selection module (manualCuration input)
- Test dataset from github.com/cma2015/iwa-miRNA Test_data directory (e.g., ath-MIR156b)
- sRNA-Seq sample read alignments supporting read stacks and expression boxplots
- miRNA–target interactions predicted by psRNAtarget

## Outputs

- Per-miRNA interactive HTML report page (e.g., ath-MIR156b_report.html) rendered via Rmarkdown
- RNA secondary structure plot with mature miRNA (5p/3p arm) locations and quality-profiling results
- Read stacks showing read support of the identified miRNA
- Boxplot of expression patterns and arm-selection events across sRNA-Seq samples
- Bipartite network of miRNA–target interactions

## How to apply

Load the selected miRNA candidate record (e.g., ath-MIR156b) from the manualCuration inputs, using the test dataset supplied in the iwa-miRNA GitHub Test_data directory. Generate the secondary structure plot showing the location of each mature miRNA (e.g., miR156b-5p and miR156b-3p arms) within the precursor sequence together with quality-profiling results, and plot the read stacks visualizing read support of the identified miRNA. Add a boxplot of expression patterns and arm-selection events across the available sRNA-Seq samples, and construct a bipartite network depicting miRNA–target interactions predicted by psRNAtarget. Render all feature views, customized by feature type and plotted in their respective styles, into an interactive HTML report page using Rmarkdown, organized within the Galaxy framework. The rationale is to support manual curation by letting annotators inspect structure, read evidence, expression, and target evidence for each candidate in one shareable page.

## Related tools

- **iwa-miRNA** (Galaxy-based framework whose manualCuration module produces summary and report pages for selected miRNA candidates) — https://github.com/cma2015/iwa-miRNA
- **Rmarkdown** (Renders the per-miRNA feature views into HTML report pages for easy visualization, interpretation, and sharing)
- **psRNAtarget** (Predicts miRNA–target interactions that are depicted as a bipartite network in the report page)
- **Galaxy** (Hosts the manualCuration module and its report-generation workflow within a user-friendly web interface)

## Examples

```
Rscript -e "rmarkdown::render('ath-MIR156b_report.Rmd', output_file='ath-MIR156b_report.html')"
```

## Evaluation signals

- ath-MIR156b_report.html exists in the output directory for the selected candidate
- The report file is a well-formed HTML document produced by Rmarkdown
- The report contains the secondary structure plot with both mature arms (miR156b-5p and miR156b-3p) located on the precursor
- The report contains read stacks, an expression/arm-selection boxplot across sRNA-Seq samples, and the psRNAtarget bipartite target-interaction network
- All feature views are rendered in their feature-type-specific styles within the Galaxy-organized report page

## Limitations

- iwa-miRNA cannot handle sRNA-Seq data from species without genome sequences
- Sequencing reads that do not map to the reference genome are ignored in the current version
- Downstream analysis of selected miRNAs is limited (no GO annotation, miRNA set enrichment, or cross-species comparative analysis)
- Wet-lab experiments are still required to validate miRNA candidates of particular interest before functional investigation

## Evidence

- [methods] The iwa-miRNA report page generated for each selected miRNA (e.g., the ath-MIR156b page) includes RNA secondary structure, read support of miRNA, miRNA arm expression patterns, arm selection events across different samples, and miRNA–target interactions.: [evidence span withheld — anchor status: not_found]
- [methods] summarizes the results into HTML pages, using Rmarkdown for easy visualization, interpretation, and sharing.: "summarizes the results into HTML pages, using Rmarkdown for easy visualization, interpretation, and sharing."
- [methods] A bipartite network is constructed to depict miRNA–target interactions predicted by psRNAtarget.: "A bipartite network is constructed to depict miRNA–target interactions predicted by psRNAtarget [36]."
- [intro] The source codes of iwa-miRNA have been combined into a Galaxy platform, organized with user-friendly web interfaces: "The source codes of iwa-miRNA have been combined into a Galaxy platform, organized with user-friendly web interfaces"
- [readme] Test datasets referred in the tutorials for iwa-miRNA: https://github.com/cma2015/iwa-miRNA/tree/master/Test_data: "Test datasets referred in the tutorials for iwa-miRNA: https://github.com/cma2015/iwa-miRNA/tree/master/Test_data"
