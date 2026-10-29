---
name: gcn-topological-assessment
description: Use when a previously inferred gene co-expression network (GCN) needs
  its performance measured via topological assessment as part of the GNIAP overlap/topological/biological
  evaluation pipeline.
when_to_use_negative:
- The GCN has not yet been inferred — run the prior GNIAP inference sub-task first,
  since this skill loads the network produced by that step
- The question only concerns agreement with reference co-expression edges or biological
  module relevance in isolation — those are the separate overlap analysis and biological
  assessment dimensions of the GNIAP workflow
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0602
tools:
- name: GNIAP
  role: R package whose topological assessment function computes network-level topological
    properties of the inferred GCN
  repo: https://github.com/ozgurcingiz/GNIAP
- name: R
  role: Runtime environment in which GNIAP and its dependent packages are installed
    and the assessment workflow is executed
  repo: https://github.com/ozgurcingiz/GNIAP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s12539-021-00440-9
    title: The Performance Comparison of Gene Co-expression Networks of Breast and
      Prostate Cancer using Different Selection Criteria
  source_content_hash: sha256:f4a64d8c726c46daf7410e7889dd4deec9c5bdbcdcbb07eb03f77c9837d96934
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ozgurcingiz/GNIAP
  tools:
  - GNIAP
  - R
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gcn-topological-assessment

## Summary

Assessment stage of the GNIAP R package workflow in which an inferred gene co-expression network (GCN) is evaluated for its network-level topological properties, as one of three combined performance dimensions (overlap analysis, topological assessment, and biological assessment) described in the GNIAP R Package Reproducibility Document.

## When to use

Use when a previously inferred gene co-expression network (GCN) needs its performance measured via topological assessment as part of the GNIAP overlap/topological/biological evaluation pipeline.

## When NOT to use

- The GCN has not yet been inferred — run the prior GNIAP inference sub-task first, since this skill loads the network produced by that step
- The question only concerns agreement with reference co-expression edges or biological module relevance in isolation — those are the separate overlap analysis and biological assessment dimensions of the GNIAP workflow

## Inputs

- Inferred gene co-expression network (GCN) from the prior GNIAP inference sub-task
- GNIAP R package installed with its dependent packages (per GNIAP Installation Document)
- GNIAP R Package Reproducibility Document (data files and instructions from ozgurcingiz/GNIAP)

## Outputs

- Network-level topological assessment results for the GCN
- Consolidated assessment metrics table (assessment_metrics.csv) combining overlap, topological, and biological assessment results

## How to apply

Install the GNIAP R package and its dependent packages following the GNIAP Installation Document in the ozgurcingiz/GNIAP repository. Load the inferred GCN produced by the prior inference sub-task together with the instructions in the GNIAP R Package Reproducibility Document, which displays all data files used in the study. Run the GNIAP topological assessment function to compute network-level topological properties of the GCN. Combine the resulting topological metrics with the overlap analysis results (agreement with reference co-expression edges) and the biological assessment results (relevance of GCN modules) into a single assessment metrics table written to assessment_metrics.csv. Judging correctness: the topological results must be reported alongside — not instead of — the overlap and biological assessments, since the reproducibility document defines GCN performance in terms of all three dimensions combined.

## Related tools

- **GNIAP** (R package whose topological assessment function computes network-level topological properties of the inferred GCN) — https://github.com/ozgurcingiz/GNIAP
- **R** (Runtime environment in which GNIAP and its dependent packages are installed and the assessment workflow is executed) — https://github.com/ozgurcingiz/GNIAP

## Evaluation signals

- Topological assessment results are produced by a GNIAP function run on the GCN loaded from the prior inference sub-task
- assessment_metrics.csv contains topological metrics consolidated together with overlap analysis and biological assessment results, not topological metrics alone
- The installed GNIAP R package matches the installation and dependent-package procedure in the GNIAP Installation Document
- The reproducibility run reproduces the assessment on the same data files displayed in the GNIAP R Package Reproducibility Document

## Limitations

- No discussion-section text was available for the article, so caveats specific to the topological metrics themselves could not be traced
- The topological dimension is only one of three combined performance dimensions; using it alone gives an incomplete GCN performance measure per the GNIAP reproducibility document
- The reproducibility document is distributed as a .docx in the GitHub repository rather than as executable code, so exact function signatures and thresholds must be read from that document

## Evidence

- [intro] Three combined performance dimensions: "It also contains how researchers infer gene co-expression networks (GCNs) and measure the performance of GCNs in terms of overlap analysis, topological and biological assessment."
- [intro] Installation prerequisite: "This document presents how users install GNIAP and its dependent packages."
- [readme] Package identity: "Gene Co-expression Network Inference and Assessment Package"
