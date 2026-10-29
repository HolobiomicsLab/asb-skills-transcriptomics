---
name: co-expression-network-performance-measurement
description: Use when an inferred GCN (from a prior GNIAP inference sub-task) needs
  its performance quantified across overlap with reference edges, network topology,
  and biological relevance of modules.
when_to_use_negative:
- No inferred GCN or reference co-expression edges are available yet — run the GNIAP
  inference sub-task first
- The goal is GCN inference itself rather than performance measurement of an existing
  network
- The analysis targets a tool other than the GNIAP R package workflow (its functions
  define the overlap, topological, and biological assessment steps)
edam_operation: http://edamontology.org/operation_0277
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: GNIAP
  role: R package providing the overlap analysis, topological assessment, and biological
    assessment functions used to measure GCN performance
  repo: https://github.com/ozgurcingiz/GNIAP
- name: R
  role: Runtime environment in which the GNIAP package and its assessment functions
    are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s12539-021-00440-9
    title: The Performance Comparison of Gene Co-expression Networks of Breast and
      Prostate Cancer using Different Selection Criteria
  source_content_hash: sha256:818003790afdb6d5cf28c989ccf9e06cdddfc52f9e3090fa1d387569edd82d67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ozgurcingiz/GNIAP
  tools:
  - GNIAP
  - R
  edam_operation: http://edamontology.org/operation_0277
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# co-expression-network-performance-measurement

## Summary

Measures the performance of an inferred gene co-expression network (GCN) using the GNIAP R package along three combined dimensions: overlap analysis, topological assessment, and biological assessment of modules.

## When to use

Use when an inferred GCN (from a prior GNIAP inference sub-task) needs its performance quantified across overlap with reference edges, network topology, and biological relevance of modules.

## When NOT to use

- No inferred GCN or reference co-expression edges are available yet — run the GNIAP inference sub-task first
- The goal is GCN inference itself rather than performance measurement of an existing network
- The analysis targets a tool other than the GNIAP R package workflow (its functions define the overlap, topological, and biological assessment steps)

## Inputs

- Inferred gene co-expression network (GCN) produced by a prior GNIAP inference sub-task
- Reference co-expression edges used for overlap analysis
- GNIAP R package installation and reproducibility instructions from the ozgurcingiz/GNIAP repository (GNIAP Installation Document.docx, GNIAP R Package Reproducibility Document.docx)

## Outputs

- Overlap analysis results quantifying agreement between inferred GCN and reference co-expression edges
- Network-level topological assessment metrics
- Biological assessment results evaluating the biological relevance of GCN modules
- Consolidated assessment metrics table written to assessment_metrics.csv

## How to apply

Install GNIAP and its dependent packages by following the GNIAP Installation Document in the ozgurcingiz/GNIAP repository, and consult the GNIAP R Package Reproducibility Document for the data files and steps used in the study. Load the previously inferred GCN together with the reference co-expression edges, then run the GNIAP overlap analysis function to quantify agreement between the inferred GCN and reference co-expression edges. Next, run the GNIAP topological assessment function to compute network-level topological properties, and run the GNIAP biological assessment function to evaluate the biological relevance of the GCN modules. Consolidate the overlap, topological, and biological assessment results into a single metrics table written to assessment_metrics.csv, following the reproduction workflow described for the package.

## Related tools

- **GNIAP** (R package providing the overlap analysis, topological assessment, and biological assessment functions used to measure GCN performance) — https://github.com/ozgurcingiz/GNIAP
- **R** (Runtime environment in which the GNIAP package and its assessment functions are executed)

## Evaluation signals

- All three assessment dimensions (overlap analysis, topological assessment, biological assessment) have been run and produced results
- Results from the three assessments are consolidated into a single assessment metrics table
- The consolidated table is written to assessment_metrics.csv
- The overlap analysis quantifies agreement between the inferred GCN and the reference co-expression edges rather than arbitrary statistics
- The biological assessment specifically evaluates the biological relevance of GCN modules

## Limitations

- Concrete GNIAP function signatures, parameters, and thresholds live in external .docx documents (Installation Document, Reproducibility Document) in the repository, not in the article text or README
- The reproducibility document depends on specific data files used in the authors' study; other datasets may require adaptation
- The article provides no discussion-section text with additional caveats (no changelog found), so edge cases are undocumented

## Evidence

- [intro] Three assessment dimensions in GNIAP: "It also contains how researchers infer gene co-expression networks (GCNs) and measure the performance of GCNs in terms of overlap analysis, topological and biological assessment."
- [readme] GNIAP installation purpose: "This document presents how users install GNIAP and its dependent packages."
- [readme] Reproducibility document scope: "This document display all data files we used in our study. It also contains how researchers infer gene co-expression networks (GCNs) and measure the performance of GCNs"
- [other] Assessment steps from source card workflow: [evidence span withheld — anchor status: not_found]
