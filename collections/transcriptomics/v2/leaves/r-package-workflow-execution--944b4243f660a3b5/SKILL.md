---
name: r-package-workflow-execution
description: Use when an inferred gene co-expression network (GCN) from expression
  data must be installed, reproduced, or assessed via overlap analysis, topological
  assessment, and biological assessment using the GNIAP R package.
when_to_use_negative:
- The goal is GCN inference from raw expression data alone with no performance assessment
  — use the GNIAP inference sub-task instead of the assessment workflow
- The package installation document and reproducibility document are unavailable,
  since the skill depends on GNIAP's documented installation and reproduction steps
- The task targets a different R package's workflow that does not implement overlap,
  topological, and biological assessment
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GNIAP
  role: 'R package providing the GCN inference and assessment workflow: overlap analysis,
    topological assessment, and biological assessment functions'
  repo: https://github.com/ozgurcingiz/GNIAP
- name: R
  role: Runtime environment in which the GNIAP package is installed and its assessment
    workflow is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s12539-021-00440-9
    title: The Performance Comparison of Gene Co-expression Networks of Breast and
      Prostate Cancer using Different Selection Criteria
  source_content_hash: sha256:7342880bdd1e40829117d4f892cbba9df6bbcc4fbf089ae53c4ce875460c945d
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
yes_rate: 0.6
n_claims: 5
---

# r-package-workflow-execution

## Summary

Executes the GNIAP R package (Gene Co-expression Network Inference and Assessment Package) workflow from the ozgurcingiz/GNIAP repository: installing the package and its dependencies, inferring a gene co-expression network (GCN) from expression data, and measuring GCN performance in terms of overlap analysis, topological assessment, and biological assessment.

## When to use

Use when an inferred gene co-expression network (GCN) from expression data must be installed, reproduced, or assessed via overlap analysis, topological assessment, and biological assessment using the GNIAP R package.

## When NOT to use

- The goal is GCN inference from raw expression data alone with no performance assessment — use the GNIAP inference sub-task instead of the assessment workflow
- The package installation document and reproducibility document are unavailable, since the skill depends on GNIAP's documented installation and reproduction steps
- The task targets a different R package's workflow that does not implement overlap, topological, and biological assessment

## Inputs

- Inferred gene co-expression network (GCN) from the prior GNIAP inference sub-task
- Reference co-expression edges for overlap analysis
- GNIAP R package and its dependent packages (per GNIAP Installation Document)
- GNIAP R Package Reproducibility Document (data files and step-by-step workflow)

## Outputs

- Overlap analysis metrics quantifying agreement between inferred GCN and reference edges
- Network-level topological assessment properties of the GCN
- Biological assessment results evaluating relevance of GCN modules
- assessment_metrics.csv consolidating overlap, topological, and biological assessment results

## How to apply

Install GNIAP and its dependent packages by following the GNIAP Installation Document linked in the repository README, which 'presents how users install GNIAP and its dependent packages.' Then consult the GNIAP R Package Reproducibility Document, which 'display[s] all data files we used in our study' and contains the exact inference and assessment steps. Load the inferred GCN produced by the prior inference sub-task plus the reference co-expression data, run the GNIAP overlap analysis function to quantify agreement between inferred GCN edges and reference co-expression edges, run the topological assessment function to compute network-level topological properties, and run the biological assessment function to evaluate the biological relevance of the GCN modules. Consolidate the overlap, topological, and biological results into a single assessment metrics table written to assessment_metrics.csv, so all three performance dimensions are reported together as in the reproducibility document.

## Related tools

- **GNIAP** (R package providing the GCN inference and assessment workflow: overlap analysis, topological assessment, and biological assessment functions) — https://github.com/ozgurcingiz/GNIAP
- **R** (Runtime environment in which the GNIAP package is installed and its assessment workflow is executed)

## Examples

```
Rscript -e "library(GNIAP)"  # after following GNIAP Installation Document; then run GNIAP's overlap analysis, topological assessment, and biological assessment functions per the Reproducibility Document
```

## Evaluation signals

- assessment_metrics.csv exists and contains rows/columns covering all three dimensions: overlap analysis, topological assessment, and biological assessment
- The overlap analysis output quantifies agreement between inferred GCN edges and reference co-expression edges (e.g., an overlap count or proportion of shared edges)
- The topological assessment output contains network-level properties (not module-level or gene-level metrics only)
- The biological assessment output evaluates the relevance of GCN modules, not the whole network alone
- The executed steps reproduce the sequence described in the GNIAP R Package Reproducibility Document using the same data files listed there

## Limitations

- The README provides only document links (Installation Document and Reproducibility Document as .docx files); the detailed function names and parameter values must be read from those documents rather than the README itself
- The assessment depends on a GCN already inferred by a prior sub-task; the assessment workflow alone does not infer the network
- Reproducibility relies on the exact data files used in the study, which are listed in the Reproducibility Document

## Evidence

- [readme] This document presents how users install GNIAP and its dependent packages.: "This document presents how users install GNIAP and its dependent packages."
- [intro] GNIAP supports inferring GCNs and measuring performance via overlap, topological, and biological assessment: "It also contains how researchers infer gene co-expression networks (GCNs) and measure the performance of GCNs in terms of overlap analysis, topological and biological assessment."
- [readme] Reproducibility document displays all data files used in the study: "This document display all data files we used in our study."
- [intro] GNIAP is the Gene Co-expression Network Inference and Assessment Package: "GNIAP
Gene Co-expression Network Inference and Assessment Package"
