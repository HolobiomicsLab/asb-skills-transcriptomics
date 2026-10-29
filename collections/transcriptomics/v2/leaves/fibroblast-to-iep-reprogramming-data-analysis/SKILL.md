---
name: fibroblast-to-iep-reprogramming-data-analysis
description: Use when reprocessing the Biddy MEF-to-iEP reprogramming time course
  (or an analogous fibroblast-to-iEP scRNA-seq time course) to reproduce Figure 1F-style
  trajectory structure with manually annotated clusters and marker overlays.
when_to_use_negative:
- The goal is perturbation simulation or GRN inference from scratch rather than reproducing
  the PAGA-based trajectory reprocessing — use CellOracle in silico KO/overexpression
  simulation workflows instead.
- Input data is not fibroblast-to-iEP reprogramming scRNA-seq (e.g., hematopoiesis
  or other systems) — the cluster annotation and marker expectations (Apoa1/Col1a2)
  are specific to the MEF-to-iEP system.
- Only raw sequencing FASTQ is available — this skill assumes a Cell Ranger-processed
  count matrix; run Cell Ranger v6.0.1 first.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Cell Ranger 6.0.1
  role: Processed the 10x Chromium scRNA-seq data into the count matrix that is loaded
    for reprocessing
- name: CellOracle
  role: Python pipeline (Python 3.8, tested with 3.10) used for the scRNA-seq processing,
    embedding, PAGA partitioning, and force-directed graph layout with marker projections
  repo: https://github.com/morris-lab/CellOracle
- name: Capybara
  role: Cell type classification of the reprogramming states
  repo: https://github.com/morris-lab/Capybara
- name: igraph
  role: Graph library dependency for the PAGA/force-directed graph analysis
  repo: https://github.com/igraph/rigraph
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:6b4c4a2d0c0d0198d2274f602f4ea31f7a495863401560e67ea9f98561efc9e2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/morris-lab/CellOracle
  repo_url: https://github.com/morris-lab/CellOracle
  tools:
  - 10x Chromium
  - Capybara
  - Cell Ranger 6.0.1
  - CellOracle
  - igraph
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# fibroblast-to-iep-reprogramming-data-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reprocess the published 27,663-cell, 8-time-point MEF-to-iEP reprogramming scRNA-seq dataset (Cell Ranger v6.0.1, 10x Chromium) into a PAGA-partitioned force-directed graph of 15 clusters spanning fibroblast, transition, dead-end, and reprogrammed iEP states, with marker projections for Apoa1 and Col1a2.

## When to use

Use when reprocessing the Biddy MEF-to-iEP reprogramming time course (or an analogous fibroblast-to-iEP scRNA-seq time course) to reproduce Figure 1F-style trajectory structure with manually annotated clusters and marker overlays.

## When NOT to use

- The goal is perturbation simulation or GRN inference from scratch rather than reproducing the PAGA-based trajectory reprocessing — use CellOracle in silico KO/overexpression simulation workflows instead.
- Input data is not fibroblast-to-iEP reprogramming scRNA-seq (e.g., hematopoiesis or other systems) — the cluster annotation and marker expectations (Apoa1/Col1a2) are specific to the MEF-to-iEP system.
- Only raw sequencing FASTQ is available — this skill assumes a Cell Ranger-processed count matrix; run Cell Ranger v6.0.1 first.

## Inputs

- Published 27,663-cell, 8-time-point MEF-to-iEP scRNA-seq count matrix (10x Chromium, Cell Ranger v6.0.1 processed)
- Gene marker list: Apoa1 (iEP marker), Col1a2 (fibroblast marker)

## Outputs

- Force-directed graph layout with 15 manually annotated clusters grouped into five cell types (fibroblasts, early transition, transition, dead-end, reprogrammed iEPs)
- Apoa1 and Col1a2 marker expression projections onto the force-directed layout (Figure 1F-equivalent panel)

## How to apply

Load the published 27,663-cell, 8-time-point count matrix generated with the Cell Ranger v6.0.1 pipeline from 10x Chromium data. Normalize, filter, and embed the cells, then run PAGA on the neighbor graph to partition the reprogramming time course into 15 clusters, annotating them into five cell-type groups: fibroblasts, early transition, transition, dead-end, and reprogrammed iEPs. Compute a force-directed graph layout on the PAGA-partitioned single-cell graph (CellOracle ecosystem, Python 3.8/3.10, with igraph support). Project Apoa1 (iEP marker) and Col1a2 (fibroblast marker) expression onto the force-directed layout to color cells by marker signal. Optionally use Capybara for cell type classification of the reprogramming states. Validate against Figure 1F: 15 clusters present, Apoa1 enriched in iEP clusters, Col1a2 enriched in fibroblast clusters.

## Related tools

- **Cell Ranger 6.0.1** (Processed the 10x Chromium scRNA-seq data into the count matrix that is loaded for reprocessing)
- **CellOracle** (Python pipeline (Python 3.8, tested with 3.10) used for the scRNA-seq processing, embedding, PAGA partitioning, and force-directed graph layout with marker projections) — https://github.com/morris-lab/CellOracle
- **Capybara** (Cell type classification of the reprogramming states) — https://github.com/morris-lab/Capybara
- **igraph** (Graph library dependency for the PAGA/force-directed graph analysis) — https://github.com/igraph/rigraph

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('mef_iep_processed.h5ad'); sc.pp.neighbors(adata); sc.tl.paga(adata, groups='cluster'); sc.pl.paga_compare(adata, basis='fa', color=['Apoa1','Col1a2'])
```

## Evaluation signals

- The reprocessed graph contains exactly 15 clusters
- Clusters group into the five expected cell-type categories: fibroblasts, early transition, transition, dead-end, and reprogrammed iEPs
- Apoa1 signal is enriched in the iEP clusters and Col1a2 signal is enriched in the fibroblast clusters in the force-directed layout
- Cell count matches the published 27,663 cells across 8 time points
- Expert review confirms the panel reproduces Figure 1F structure

## Limitations

- Cluster annotations are manually assigned; threshold selections in Capybara-based classification can introduce variations across runs
- Reproduction fidelity is judged by expert review against the published Figure 1F, so layout and clustering stochasticity may require manual matching
- The reprocessing depends on the published count matrix having been generated with Cell Ranger v6.0.1; other aligner/quantifier versions may not recapitulate the same 15-cluster structure

## Evidence
