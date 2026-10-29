---
name: gcn-overlap-analysis
description: Use when an inferred gene co-expression network (GCN) needs its performance
  measured, and agreement with reference co-expression edges, network topology, and
  biological relevance of modules must be quantified.
when_to_use_negative:
- No inferred GCN is available yet — the GNIAP inference sub-task must be run first
  before performance assessment
- The goal is network inference rather than performance evaluation of an already-inferred
  GCN
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0605
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
  source_content_hash: sha256:083f747e22696b56a7b1bd08d5205d7953769db0eb96ba1167ecebb77565e05f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ozgurcingiz/GNIAP
  tools:
  - GNIAP
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# gcn-overlap-analysis

## Summary

Assess the performance of inferred gene co-expression networks (GCNs) using the GNIAP R package, which combines overlap analysis, topological assessment, and biological assessment of network modules.

## When to use

Use when an inferred gene co-expression network (GCN) needs its performance measured, and agreement with reference co-expression edges, network topology, and biological relevance of modules must be quantified.

## When NOT to use

- No inferred GCN is available yet — the GNIAP inference sub-task must be run first before performance assessment
- The goal is network inference rather than performance evaluation of an already-inferred GCN

## Inputs

- Inferred gene co-expression network (GCN) from a prior GNIAP inference step
- Reference co-expression edges for overlap comparison
- GNIAP R package installation and reproducibility documents (GNIAP Installation Document.docx, GNIAP R Package Reproducibility Document.docx)

## Outputs

- Overlap analysis results quantifying agreement between inferred and reference co-expression edges
- Topological assessment metrics of the GCN
- Biological assessment results for GCN modules
- Consolidated assessment metrics table (assessment_metrics.csv)

## How to apply

Load the inferred GCN produced by a prior inference sub-task together with the GNIAP R package (installed per the GNIAP Installation Document from the ozgurcingiz/GNIAP repository). Run the GNIAP overlap analysis function to quantify agreement between the inferred GCN's edges and reference co-expression edges. Run the GNIAP topological assessment function to compute network-level topological properties of the GCN, then run the GNIAP biological assessment function to evaluate the biological relevance of the GCN modules. The GNIAP R Package Reproducibility Document documents the exact data files and commands used in the authors' breast and prostate cancer GCN study, so follow it step by step for reproducibility. Finally, consolidate the overlap, topological, and biological assessment results into a single assessment metrics table and write it to assessment_metrics.csv.

## Related tools

- **GNIAP** (R package providing the overlap analysis, topological assessment, and biological assessment functions used to measure GCN performance) — https://github.com/ozgurcingiz/GNIAP
- **R** (Runtime environment in which the GNIAP package and its dependent packages are installed and executed)

## Evaluation signals

- All three assessment dimensions (overlap, topological, biological) were executed and each produced results
- The overlap analysis output quantifies agreement against reference co-expression edges rather than being empty
- Topological metrics are computed at the network level for the inferred GCN
- Biological assessment evaluates the relevance of GCN modules, not just individual genes
- A consolidated assessment_metrics.csv table contains columns/metrics from all three assessments

## Limitations

- The GNIAP reproducibility document (a .docx file in the GitHub repository) is the authoritative record of the data files used; without it, exact reproduction of the study's assessment is not possible
- GNIAP requires installation of dependent packages as described in the GNIAP Installation Document before the assessment functions can run
- No discussion-section text was available in the extracted article, so caveats beyond those documents are not documented here

## Evidence

- [intro] GNIAP supports inferring GCNs and measuring their performance via overlap analysis, topological assessment, and biological assessment: "It also contains how researchers infer gene co-expression networks (GCNs) and measure the performance of GCNs in terms of overlap analysis, topological and biological assessment."
- [readme] GNIAP R package reproducibility document displays all data files used in the study: "You can also download GNIAP R package reproducability document from the link below. This document display all data files we used in our study."
- [readme] GNIAP installation document covers installation of GNIAP and its dependent packages: "This document presents how users install GNIAP and its dependent packages."
- [intro] GNIAP is the Gene Co-expression Network Inference and Assessment Package, implemented in R: "GNIAP
Gene Co-expression Network Inference and Assessment Package"
