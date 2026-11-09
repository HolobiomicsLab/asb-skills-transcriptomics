---
name: srna-read-stack-visualization
description: Use when you need to visually inspect read stacks, expression boxplots,
  and arm-selection events for candidate plant miRNAs (e.g., ath-MIR156b) to support
  manual curation decisions.
when_to_use_negative:
- You only need genome-wide candidate lists rather than per-candidate feature-level
  curation views (use MiRNA Compilation/MiRNA Selection outputs instead)
- sRNA-Seq data come from a species without a reference genome, since unmapped reads
  are ignored and cannot support the read-stack display
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0659
tools:
- name: iwa-miRNA
  role: Framework whose manualCuration module generates the per-miRNA report pages
    with read stacks and expression boxplots
  repo: https://github.com/cma2015/iwa-miRNA
- name: Rmarkdown
  role: Renders the feature views into HTML report pages for easy visualization, interpretation,
    and sharing
- name: psRNAtarget
  role: Provides predicted miRNA–target interactions depicted in the report's bipartite
    network
- name: Galaxy
  role: Hosts the manualCuration report-generation workflow and its dependencies
  repo: https://github.com/cma2015/iwa-miRNA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:e3a700e39376d5e877851c132eee19af564360b70fc729f4db16dece08ff0a6d
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# srna-read-stack-visualization

## Summary

Visualize the sRNA-Seq read support, arm expression patterns, and arm-selection events of each selected miRNA candidate as feature views rendered into per-miRNA interactive HTML report pages within the iwa-miRNA manualCuration module.

## When to use

Use when you need to visually inspect read stacks, expression boxplots, and arm-selection events for candidate plant miRNAs (e.g., ath-MIR156b) to support manual curation decisions.

## When NOT to use

- You only need genome-wide candidate lists rather than per-candidate feature-level curation views (use MiRNA Compilation/MiRNA Selection outputs instead)
- sRNA-Seq data come from a species without a reference genome, since unmapped reads are ignored and cannot support the read-stack display

## Inputs

- Output of the MiRNA Selection module (selected miRNA candidates, e.g., ath-MIR156b)
- sRNA-Seq alignment/read data across the analyzed samples
- miRNA precursor secondary structure and quality-profiling results
- psRNAtarget-predicted miRNA–target interactions

## Outputs

- Per-miRNA interactive HTML report page (e.g., ath-MIR156b_report.html) generated with Rmarkdown
- Read-stack plot of miRNA read support
- Expression boxplot showing arm expression patterns and arm-selection events across samples
- Secondary-structure plot marking mature miRNA arm positions
- Bipartite miRNA–target interaction network

## How to apply

Load the selected miRNA candidate record (e.g., ath-MIR156b) from the manualCuration inputs, using the test datasets supplied in the iwa-miRNA GitHub Test_data directory. Generate the secondary-structure plot showing the locations of the mature miR156b-5p and miR156b-3p arms within the precursor sequence together with quality-profiling results. Plot the read stacks showing read support of the identified miRNA, plus a boxplot visualizing its expression patterns and arm-selection events across the available sRNA-Seq samples. Construct the bipartite network of miRNA–target interactions predicted by psRNAtarget, then render all feature views into an interactive HTML page using Rmarkdown, organized within the Galaxy framework.

## Related tools

- **iwa-miRNA** (Framework whose manualCuration module generates the per-miRNA report pages with read stacks and expression boxplots) — https://github.com/cma2015/iwa-miRNA
- **Rmarkdown** (Renders the feature views into HTML report pages for easy visualization, interpretation, and sharing)
- **psRNAtarget** (Provides predicted miRNA–target interactions depicted in the report's bipartite network)
- **Galaxy** (Hosts the manualCuration report-generation workflow and its dependencies) — https://github.com/cma2015/iwa-miRNA

## Examples

```
docker pull malab/iwa-mirna
```

## Evaluation signals

- ath-MIR156b_report.html exists for each selected candidate and is a well-formed HTML document
- Report page contains the required feature fields: secondary-structure plot, read stacks, expression boxplot, and target-interaction network
- Read-stack plot reflects reads mapped to the precursor with mature 5p/3p arm positions annotated on the secondary structure
- Expression boxplot displays arm-selection events consistently across the available sRNA-Seq samples
- Bipartite network nodes/edges correspond to psRNAtarget predictions for the candidate miRNA

## Limitations

- Sequencing reads that do not map to the reference genome are ignored in the current version, so read stacks only cover mappable reads
- Aggregated candidate lists contain false positives; read-stack visuals are curation aids, not validation — wet experiments are still needed before functional investigation

## Evidence

- [methods] summarizes the results into HTML pages, using Rmarkdown for easy visualization, interpretation, and sharing.: "summarizes the results into HTML pages, using Rmarkdown for easy visualization, interpretation, and sharing."
- [methods] bipartite target network: "A bipartite network is constructed to depict miRNA–target interactions predicted by psRNAtarget [36]."
- [methods] manualCuration output: [evidence span withheld — anchor status: not_found]
- [readme] module composition: "The iwa-miRNA is composed with three functional modules: MiRNA Compilation, MiRNA Selection, and Manual Curation."
- [discussion] unmapped-read limitation: "In the current version, iwa-miRNA ignores sequencing reads that do not map to the reference genome."
