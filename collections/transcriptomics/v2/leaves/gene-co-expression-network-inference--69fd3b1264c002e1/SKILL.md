---
name: gene-co-expression-network-inference
description: Use when raw gene expression data (e.g. breast or prostate cancer expression
  matrices) must be turned into a weighted gene co-expression network and the resulting
  GCN's performance needs to be assessed via overlap, topological, and biological
  criteria.
when_to_use_negative:
- The input is already an inferred co-expression network (nodes and edges) rather
  than raw expression data — proceed directly to overlap/topological/biological assessment
  instead of re-inferring.
- The goal is performance comparison across different gene selection criteria rather
  than producing a single inferred GCN for one dataset.
- The environment cannot install GNIAP and its dependent packages per the GNIAP Installation
  Document.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: GNIAP
  role: R package that performs the gene co-expression network inference and subsequent
    overlap, topological, and biological assessment
  repo: https://github.com/ozgurcingiz/GNIAP
- name: R
  role: Language/environment in which GNIAP is installed and the GCN inference workflow
    is executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s12539-021-00440-9
    title: The Performance Comparison of Gene Co-expression Networks of Breast and
      Prostate Cancer using Different Selection Criteria
  source_content_hash: sha256:64e3009a344603d3318596a8ff25c7ca2166e9fd15d2bc1ebc9efb4637c715c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ozgurcingiz/GNIAP
  tools:
  - GNIAP
  - R
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# gene-co-expression-network-inference

## Summary

Infers gene co-expression networks (GCNs) from cancer expression datasets using the GNIAP R package, following the reproducibility workflow of Cingiz et al. (2021), and exports the resulting nodes and weighted edges for downstream overlap, topological, and biological assessment.

## When to use

Use when raw gene expression data (e.g. breast or prostate cancer expression matrices) must be turned into a weighted gene co-expression network and the resulting GCN's performance needs to be assessed via overlap, topological, and biological criteria.

## When NOT to use

- The input is already an inferred co-expression network (nodes and edges) rather than raw expression data — proceed directly to overlap/topological/biological assessment instead of re-inferring.
- The goal is performance comparison across different gene selection criteria rather than producing a single inferred GCN for one dataset.
- The environment cannot install GNIAP and its dependent packages per the GNIAP Installation Document.

## Inputs

- Breast cancer gene expression data files (as listed in the GNIAP R Package Reproducibility Document)
- Prostate cancer gene expression data files (as listed in the GNIAP R Package Reproducibility Document)
- GNIAP R package installed with its dependent packages

## Outputs

- inferred_gcn.csv — inferred gene co-expression network as nodes and weighted edges

## How to apply

First install the GNIAP R package and its dependent packages by following the GNIAP Installation Document from the ozgurcingiz/GNIAP GitHub repository. Load the study's data files — the breast and prostate cancer expression data — as listed in the GNIAP R Package Reproducibility Document. Execute the GCN inference step of the GNIAP workflow on the loaded expression data using the function and parameters specified in that reproducibility document. Export the inferred gene co-expression network (its nodes and weighted edges) to inferred_gcn.csv so it can later be scored by the package's overlap analysis, topological assessment, and biological assessment routines.

## Related tools

- **GNIAP** (R package that performs the gene co-expression network inference and subsequent overlap, topological, and biological assessment) — https://github.com/ozgurcingiz/GNIAP
- **R** (Language/environment in which GNIAP is installed and the GCN inference workflow is executed)

## Examples

```
Rscript -e 'library(GNIAP); # follow GNIAP R Package Reproducibility Document to load breast/prostate expression data and run the GCN inference step'
```

## Evaluation signals

- inferred_gcn.csv exists and contains both nodes and weighted edges derived from the breast or prostate cancer expression data.
- The inferred GCN can be loaded back into GNIAP and assessed without errors by the package's overlap analysis, topological assessment, and biological assessment routines.
- The inference run used the exact function and parameters documented in the GNIAP R Package Reproducibility Document, so results are reproducible against the study's data files.
- GNIAP and its dependent packages installed cleanly per the GNIAP Installation Document before any inference was attempted.
- Node and edge counts in inferred_gcn.csv are consistent with the expression dataset used (all network nodes correspond to genes from the input expression matrix).

## Limitations

- The workflow depends on data files and exact parameters documented in a separate .docx (GNIAP R Package Reproducibility Document) rather than in the paper text itself.
- The README does not state specific thresholds or cutoffs for the overlap, topological, or biological assessment steps; those details must be retrieved from the reproducibility document.
- Evidence is limited to the README and article index — the article's discussion section text was unavailable, so caveats about performance across selection criteria could not be verified here.

## Evidence

- [readme] workflow steps 1-4: "This document presents how users install GNIAP and its dependent packages."
- [readme] assessment scope: "It also contains how researchers infer gene co-expression networks (GCNs) and measure the performance of GCNs in terms of overlap analysis, topological and biological assessment."
- [intro] data files listing: "The GNIAP reproducibility document contains how researchers infer gene co-expression networks (GCNs) and measure the performance of GCNs in terms of overlap analysis, topological and biological"
- [intro] package identity: "GNIAP
Gene Co-expression Network Inference and Assessment Package"
