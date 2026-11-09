---
name: gcn-biological-assessment
description: Use when an inferred gene co-expression network (GCN) has been produced
  (e.g. for breast or prostate cancer expression data) and its performance must be
  measured via overlap analysis, topological assessment, and biological assessment
  using GNIAP.
when_to_use_negative:
- No GCN has yet been inferred — GNIAP must first infer the GCN before performance
  can be measured via overlap, topological, and biological assessment
- The goal is only network inference, not performance assessment of an existing GCN
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GNIAP
  role: R package providing the overlap analysis, topological assessment, and biological
    assessment functions used to measure GCN performance
  repo: https://github.com/ozgurcingiz/GNIAP
- name: R
  role: Runtime environment in which the GNIAP package and its dependent packages
    are installed and executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s12539-021-00440-9
    title: The Performance Comparison of Gene Co-expression Networks of Breast and
      Prostate Cancer using Different Selection Criteria
  source_content_hash: sha256:2f48b913eb0c910b891d129dbd7560b5941c0edb969f8eca07797616568563dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ozgurcingiz/GNIAP
  tools:
  - GNIAP
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# gcn-biological-assessment

## Summary

Assess the performance of an inferred gene co-expression network (GCN) using the GNIAP R package, which evaluates GCNs along three combined dimensions: overlap analysis, topological assessment, and biological assessment of GCN modules.

## When to use

Use when an inferred gene co-expression network (GCN) has been produced (e.g. for breast or prostate cancer expression data) and its performance must be measured via overlap analysis, topological assessment, and biological assessment using GNIAP.

## When NOT to use

- No GCN has yet been inferred — GNIAP must first infer the GCN before performance can be measured via overlap, topological, and biological assessment
- The goal is only network inference, not performance assessment of an existing GCN

## Inputs

- Inferred gene co-expression network (GCN) from a prior GNIAP inference step
- Reference co-expression edges for overlap analysis
- Expression data files listed in the GNIAP R Package Reproducibility Document

## Outputs

- Overlap analysis metrics between the inferred GCN and reference edges
- Network-level topological assessment metrics
- Biological relevance assessment of GCN modules
- Consolidated assessment_metrics.csv file

## How to apply

Install the GNIAP R package and its dependent packages following the GNIAP Installation Document from the ozgurcingiz/GNIAP repository, then consult the GNIAP R Package Reproducibility Document, which lists all data files used in the study and the assessment procedure. Load the previously inferred GCN as the input to the assessment workflow. Run the GNIAP overlap analysis function to quantify agreement between the inferred GCN and reference co-expression edges, run the GNIAP topological assessment function to compute network-level topological properties, and run the GNIAP biological assessment function to evaluate the biological relevance of the GCN modules. Consolidate the overlap, topological, and biological assessment results into a single assessment metrics table and write it to assessment_metrics.csv. Correct application is judged by all three assessment dimensions (overlap, topological, biological) being present in the consolidated metrics table.

## Related tools

- **GNIAP** (R package providing the overlap analysis, topological assessment, and biological assessment functions used to measure GCN performance) — https://github.com/ozgurcingiz/GNIAP
- **R** (Runtime environment in which the GNIAP package and its dependent packages are installed and executed)

## Evaluation signals

- assessment_metrics.csv exists and contains columns/values from all three assessment dimensions: overlap analysis, topological assessment, and biological assessment
- Overlap analysis results quantify agreement between the inferred GCN and reference co-expression edges
- Topological assessment reports network-level topological properties of the GCN
- Biological assessment evaluates the biological relevance of each GCN module
- The run follows the exact data files listed in the GNIAP R Package Reproducibility Document, enabling reproducibility against the study's results

## Limitations

- The README documents installation of GNIAP and its dependent packages as a prerequisite; assessment cannot run without a completed installation
- The reproducibility document is distributed as a .docx file (GNIAP R Package Reproducibility Document.docx), which must be consulted for the study's data files and procedure
- No discussion-section text was available in the extracted article, so caveats specific to the assessment's interpretation could not be traced from the paper's discussion

## Evidence

- [readme] README assessment workflow: "It also contains how researchers infer gene co-expression networks (GCNs) and measure the performance of GCNs in terms of"
- [readme] README installation prerequisite: "This document presents how users install GNIAP and its dependent packages."
- [intro] Article finding on GCN performance dimensions: [evidence span withheld — anchor status: not_found]
