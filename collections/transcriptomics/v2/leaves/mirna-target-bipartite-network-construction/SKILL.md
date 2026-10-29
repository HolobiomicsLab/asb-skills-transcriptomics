---
name: mirna-target-bipartite-network-construction
description: Use when curating a selected plant miRNA candidate (e.g., ath-MIR156b)
  in iwa-miRNA and you need to visualize its predicted target interactions as a bipartite
  network on the manualCuration HTML report page.
when_to_use_negative:
- Input sRNA-Seq data are from a species without reference genome sequences — iwa-miRNA
  cannot handle such data in the current version
- The goal is downstream enrichment or cross-species comparative analysis of miRNAs
  rather than per-candidate target-interaction visualization
edam_operation: http://edamontology.org/operation_3792
edam_topics:
- http://edamontology.org/topic_0659
tools:
- name: iwa-miRNA
  role: Galaxy-based framework whose manualCuration module builds the bipartite miRNA–target
    network and renders it into the per-miRNA Rmarkdown HTML report page
  repo: https://github.com/cma2015/iwa-miRNA
- name: psRNAtarget
  role: Predicts the miRNA–target interactions that are depicted as edges of the bipartite
    network
- name: Rmarkdown
  role: Renders the network and other feature views into an interactive HTML report
    page
- name: Galaxy
  role: Framework organizing the manualCuration workflow and its dependencies
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:f8ee9c0b95ff3601102054631f62b57852a3e54ea9b1040267c0d7bc81a4e15a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Galaxy
  - Rmarkdown
  - iwa-miRNA
  - psRNAtarget
  edam_operation: http://edamontology.org/operation_3792
  edam_topics:
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mirna-target-bipartite-network-construction

## Summary

Within the iwa-miRNA Manual Curation module, a bipartite network is constructed to depict miRNA–target interactions predicted by psRNAtarget, and is rendered as one of the feature views on the per-miRNA Rmarkdown HTML report page for manual curation.

## When to use

Use when curating a selected plant miRNA candidate (e.g., ath-MIR156b) in iwa-miRNA and you need to visualize its predicted target interactions as a bipartite network on the manualCuration HTML report page.

## When NOT to use

- Input sRNA-Seq data are from a species without reference genome sequences — iwa-miRNA cannot handle such data in the current version
- The goal is downstream enrichment or cross-species comparative analysis of miRNAs rather than per-candidate target-interaction visualization

## Inputs

- Selected miRNA candidates (output of the MiRNA Selection module, e.g., ath-MIR156b record)
- Genome sequences and annotation formatted by genomePrepare
- Test dataset from github.com/cma2015/iwa-miRNA Test_data directory

## Outputs

- Bipartite network depicting miRNA–target interactions predicted by psRNAtarget
- Per-miRNA interactive HTML report page (e.g., ath-MIR156b_report.html) rendered with Rmarkdown containing the target-interaction network view

## How to apply

Load the selected miRNA candidate record (e.g., ath-MIR156b) from the manualCuration inputs, using the test dataset supplied in the iwa-miRNA GitHub Test_data directory. Run psRNAtarget to predict miRNA–target interactions for the candidate's mature sequences against the prepared genome transcript set. Construct a bipartite network whose two node classes are miRNAs and predicted targets, with edges representing predicted interactions. Render the network as one feature view of the per-miRNA Rmarkdown HTML report page, organized within the Galaxy framework. Validate that the report page contains the target-interaction network alongside the secondary structure plot, read stacks, and expression boxplot.

## Related tools

- **iwa-miRNA** (Galaxy-based framework whose manualCuration module builds the bipartite miRNA–target network and renders it into the per-miRNA Rmarkdown HTML report page) — https://github.com/cma2015/iwa-miRNA
- **psRNAtarget** (Predicts the miRNA–target interactions that are depicted as edges of the bipartite network)
- **Rmarkdown** (Renders the network and other feature views into an interactive HTML report page)
- **Galaxy** (Framework organizing the manualCuration workflow and its dependencies)

## Evaluation signals

- ath-MIR156b_report.html exists and is a well-formed HTML document
- The report page contains the required feature fields, including the target-interaction network
- The network is bipartite: nodes partition into miRNA and target classes, with edges only between the two classes
- Predicted edges correspond to miRNA–target interactions returned by psRNAtarget
- The report page renders alongside the secondary structure plot, read stacks, and expression boxplot

## Limitations

- Interactions are computationally predicted by psRNAtarget and require wet experiments to validate before functional investigation
- iwa-miRNA ignores sequencing reads that do not map to the reference genome, limiting candidates for species with incomplete references
- The aggregated candidate list certainly contains false positive hits

## Evidence

- [methods] A bipartite network is constructed to depict miRNA–target interactions predicted by psRNAtarget: "A bipartite network is constructed to depict miRNA–target interactions predicted by psRNAtarget [36]."
- [methods] manualCuration produces summary and report pages: [evidence span withheld — anchor status: not_found]
- [methods] Rmarkdown HTML report for easy visualization: "summarizes the results into HTML pages, using Rmarkdown for easy visualization, interpretation, and sharing."
- [other] Manual curation report includes target interactions among feature views: [evidence span withheld — anchor status: not_found]
- [readme] README: iwa-miRNA modules include Manual Curation: "The iwa-miRNA is composed with three functional modules: MiRNA Compilation, MiRNA Selection, and Manual Curation."
