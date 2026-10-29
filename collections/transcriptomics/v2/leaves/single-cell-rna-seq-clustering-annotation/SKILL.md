---
name: single-cell-rna-seq-clustering-annotation
description: Use when a published 10x Chromium MEF-to-iEP reprogramming scRNA-seq
  count matrix must be reprocessed into PAGA clusters with marker-based annotation
  of fibroblast, transition, dead-end, and reprogrammed iEP states.
when_to_use_negative:
- The input is already a annotated cluster/feature table rather than a raw 10x Chromium
  count matrix requiring PAGA partitioning
- The analysis goal is in silico TF perturbation simulation or GRN inference (CellOracle
  perturbation/KO simulation) rather than reproducing the clustering and marker-projection
  figure
- The dataset is not the fibroblast-to-iEP reprogramming time course, so the fibroblast/iEP/dead-end
  cluster annotation scheme does not apply
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cell Ranger 6.0.1
  role: Pipeline used to process the raw 10x Chromium data into the count matrix that
    is clustered
  repo: https://support.10xgenomics.com/single-cell-gene-expression/software/downloads/latest
- name: CellOracle
  role: Python (3.8/3.10) toolkit providing single-cell graph analysis used to partition
    the reprogramming time course into clusters and compute layouts
  repo: https://github.com/morris-lab/CellOracle
- name: Capybara
  role: R package used for cell type classification / measuring cell identity and
    fate transitions at single-cell resolution
  repo: https://github.com/morris-lab/Capybara
- name: 10x Chromium
  role: Platform on which the single-cell libraries were prepared, producing the input
    data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:068bb29cc7b71c98ff43edf3a64a531072ebe2678eaba47eaacfc3cadcfcdbfd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/morris-lab/CellOracle
  tools:
  - 10x Chromium
  - Capybara
  - Cell Ranger
  - Cell Ranger 6.0.1
  - CellOracle
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# single-cell-rna-seq-clustering-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reprocess the 27,663-cell, 8-time-point Biddy MEF-to-iEP reprogramming scRNA-seq dataset by normalizing, embedding, running PAGA to partition cells into 15 manually annotated clusters, and rendering a force-directed graph with Apoa1/Col1a2 marker projections that recapitulates Figure 1F.

## When to use

Use when a published 10x Chromium MEF-to-iEP reprogramming scRNA-seq count matrix must be reprocessed into PAGA clusters with marker-based annotation of fibroblast, transition, dead-end, and reprogrammed iEP states.

## When NOT to use

- The input is already a annotated cluster/feature table rather than a raw 10x Chromium count matrix requiring PAGA partitioning
- The analysis goal is in silico TF perturbation simulation or GRN inference (CellOracle perturbation/KO simulation) rather than reproducing the clustering and marker-projection figure
- The dataset is not the fibroblast-to-iEP reprogramming time course, so the fibroblast/iEP/dead-end cluster annotation scheme does not apply

## Inputs

- 27,663-cell, 8-time-point MEF-to-iEP scRNA-seq count matrix processed with Cell Ranger v6.0.1 (10x Chromium platform data)
- Marker gene list: Apoa1 (iEP marker) and Col1a2 (fibroblast marker)
- Reference data for cell type classification via Capybara (single-cell reference atlas plus sample count matrix and metadata)

## Outputs

- Force-directed graph of 15 PAGA clusters grouped into five cell types (fibroblasts, early transition, transition, dead-end, reprogrammed iEPs)
- Marker expression projections of Apoa1 and Col1a2 colored onto the force-directed layout (Figure 1F equivalent)
- Manually annotated cluster identity assignments per cell

## How to apply

Load the 27,663-cell, 8-time-point count matrix generated with the Cell Ranger v6.0.1 pipeline from 10x Chromium platform data. Normalize and filter the cells, build a neighbor graph, and run PAGA to partition the reprogramming time course into 15 clusters, then manually annotate cluster identities (fibroblast, early transition, transition, dead-end, reprogrammed iEPs). Compute a force-directed graph layout on the PAGA-partitioned single-cell graph and project Apoa1 (iEP marker) and Col1a2 (fibroblast marker) expression onto the layout. The workflow uses CellOracle's Python tooling for graph analysis and Capybara (R >= 3.5.0) for cell type classification; expert review against the published Figure 1F panel validates the result.

## Related tools

- **Cell Ranger 6.0.1** (Pipeline used to process the raw 10x Chromium data into the count matrix that is clustered) — https://support.10xgenomics.com/single-cell-gene-expression/software/downloads/latest
- **CellOracle** (Python (3.8/3.10) toolkit providing single-cell graph analysis used to partition the reprogramming time course into clusters and compute layouts) — https://github.com/morris-lab/CellOracle
- **Capybara** (R package used for cell type classification / measuring cell identity and fate transitions at single-cell resolution) — https://github.com/morris-lab/Capybara
- **10x Chromium** (Platform on which the single-cell libraries were prepared, producing the input data)

## Examples

```
library("devtools"); devtools::install_github("morris-lab/Capybara"); library("Capybara")
```

## Evaluation signals

- Exactly 15 clusters are present in the PAGA partition of the 27,663-cell dataset
- Clusters group into five annotated cell types: fibroblasts, early transition, transition, dead-end, and reprogrammed iEPs
- Apoa1 expression signal is enriched in iEP clusters while Col1a2 signal is enriched in fibroblast clusters on the force-directed layout
- The rendered panel reproduces the structure of published Figure 1F upon expert review
- Capybara classification runs within its supported environment (R >= 3.5.0 with limma, impute, AnnotationDbi, GO.db, preprocessCore installed)

## Limitations

- Cluster identities are manually annotated by expert review against the published figure, so the annotation step is subjective and label definitions may vary between analysts
- Reproduction comparisons can show variations due to differences in threshold selections, as noted in the Capybara reproducibility documentation
- The 15-cluster scheme is tied to this specific Biddy MEF-to-iEP dataset and does not transfer directly to other reprogramming or tissue datasets

## Evidence

- [readme] Capybara threshold caveat: [evidence span withheld — anchor status: external_doc]
- [other] PAGA workflow on reprogramming time course: [evidence span withheld — anchor status: not_found]
