---
name: celltag-lineage-tracing-data-analysis
description: Use when analyzing scRNA-seq data from CellTag-barcoded reprogramming
  experiments where early cell states must be linked to later conversion outcomes
  via clonal lineage tracing and GRN comparison.
when_to_use_negative:
- When no lineage barcode (CellTag) data exists, since fate groups cannot be linked
  to early states clonally — the permutation test of clone membership is undefined
- When the goal is perturbation simulation or novel-factor discovery (in silico TF
  KO) rather than state-fate clonal comparison, CellOracle's perturbation module is
  the appropriate skill instead
- When the dataset is not a reprogramming time course with fibroblast-origin cells;
  Capybara reference states and the day-4 fate split are specific to this design
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3170
tools:
- name: Cell Ranger
  role: Process 10x Chromium scRNA-seq reads (v6.0.1) into the digital gene expression
    matrix
  repo: https://support.10xgenomics.com/single-cell-gene-expression/software/downloads/latest
- name: newCloneCalling
  role: Allowlist CellTag barcodes from the plasmid library and call clones from integrated
    barcode data
  repo: https://github.com/morris-lab/newCloneCalling
- name: Capybara
  role: Classify cells along the reprogramming time course to assign day-4 states
    and iEP conversion outcomes
  repo: https://github.com/morris-lab/Capybara
- name: CellOracle
  role: Build sample-specific GRN/Links objects for each day-4 state group and native
    fibroblasts
  repo: https://github.com/morris-lab/CellOracle
- name: igraph
  role: Compute TF degree centrality on inferred GRN graphs for fate-group comparison
  repo: https://github.com/igraph/rigraph
- name: Python
  role: Runtime for CellOracle GRN construction and downstream analysis (developed
    with Python 3.8, tested with 3.10)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:0e8d4e88a8ecc259c556a7a65f0558f6d53abf3d1fdd1e0abcd31679dde83274
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/morris-lab/CellOracle
  tools:
  - Capybara
  - Cell Ranger
  - CellOracle
  - Python
  - igraph
  - newCloneCalling
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# celltag-lineage-tracing-data-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Recover clonal lineages from CellTag barcoded single-cell reprogramming data and link early GRN states to fate outcomes, combining newCloneCalling clone recovery, Capybara identity classification, permutation-based fate grouping, and CellOracle/igraph degree-centrality comparison of fate-destined clones.

## When to use

Use when analyzing scRNA-seq data from CellTag-barcoded reprogramming experiments where early cell states must be linked to later conversion outcomes via clonal lineage tracing and GRN comparison.

## When NOT to use

- When no lineage barcode (CellTag) data exists, since fate groups cannot be linked to early states clonally — the permutation test of clone membership is undefined
- When the goal is perturbation simulation or novel-factor discovery (in silico TF KO) rather than state-fate clonal comparison, CellOracle's perturbation module is the appropriate skill instead
- When the dataset is not a reprogramming time course with fibroblast-origin cells; Capybara reference states and the day-4 fate split are specific to this design

## Inputs

- 10x Chromium scRNA-seq FASTQ/BCL data from a CellTag-barcoded MEF-to-iEP reprogramming time course (24,799 cells)
- Integrated CellTag barcode (lineage-barcode) data from the CellTag-multi libraries
- Reference transcriptome for Cell Ranger v6.0.1 processing
- Base-GRN generated from the Mouse sci-ATAC-seq Atlas (for CellOracle GRN inference)
- Reference cell-type datasets for Capybara identity classification

## Outputs

- Digital gene expression matrix from Cell Ranger v6.0.1
- Set of called clones (1,158 clones) with cell-to-clone assignments
- Capybara cell-identity/state classifications along the reprogramming time course
- Day-4 state-fate groups: iEP-enriched (55 cells / 9 clones) and iEP-depleted (50 cells / 43 clones)
- Sample-specific CellOracle GRN/Links objects per state group and native fibroblasts
- TF degree-centrality comparison plot (Figure 2H) separating fate-destined clone groups

## How to apply

Process the 10x Chromium scRNA-seq data (24,799-cell MEF-to-iEP reprogramming time course) with Cell Ranger v6.0.1 to obtain the digital gene expression matrix, then run the morris-lab/newCloneCalling pipeline on the integrated CellTag barcode data to call clones (expected: 1,158 clones). Classify cells along the time course with Capybara (R >= 3.5.0) to assign day-4 reprogramming states and iEP conversion outcomes. Apply a randomized (permutation) test of clone membership among successfully converted cells to split day-4 cells into iEP-enriched (55 cells / 9 clones) and iEP-depleted (50 cells / 43 clones) groups. Build sample-specific CellOracle GRN/Links objects for each day-4 state group and for native fibroblasts, compute TF degree centrality with igraph, and compare reprogrammed-destined versus dead-end-destined clones relative to native fibroblasts (Figure 2H). Rationale: sister cells follow the same reprogramming trajectories, so clone-level fate assignment lets early network state be attributed to outcome rather than batch or sampling noise.

## Related tools

- **Cell Ranger** (Process 10x Chromium scRNA-seq reads (v6.0.1) into the digital gene expression matrix) — https://support.10xgenomics.com/single-cell-gene-expression/software/downloads/latest
- **newCloneCalling** (Allowlist CellTag barcodes from the plasmid library and call clones from integrated barcode data) — https://github.com/morris-lab/newCloneCalling
- **Capybara** (Classify cells along the reprogramming time course to assign day-4 states and iEP conversion outcomes) — https://github.com/morris-lab/Capybara
- **CellOracle** (Build sample-specific GRN/Links objects for each day-4 state group and native fibroblasts) — https://github.com/morris-lab/CellOracle
- **igraph** (Compute TF degree centrality on inferred GRN graphs for fate-group comparison) — https://github.com/igraph/rigraph
- **Python** (Runtime for CellOracle GRN construction and downstream analysis (developed with Python 3.8, tested with 3.10))

## Examples

```
cellranger count --id reprog_d4 --transcriptome mm10 --fastqs fastqs/ ; Rscript newCloneCalling/call_clones.R --barcodes integrated_tags.tsv ; Capybara::classify(query, ref, map=TRUE)
```

## Evaluation signals

- Clone recovery count matches the reported 1,158 clones from the newCloneCalling pipeline
- State-group sizes match: 9 iEP-enriched clones (55 cells) and 43 iEP-depleted clones (50 cells)
- Figure 2H plot reproduces the reported degree-centrality separation between reprogrammed-destined and dead-end-destined day-4 clones
- Direction of TF connectivity differences matches: Klf6, Tbx5, Tfapb2, and Foxs1 more highly connected in failing cells; Fos, Klf5, and Junb more highly connected in iEP-destined cells
- Capybara classification yields day-4 state assignments and iEP conversion outcomes consistent with the reprogramming time course

## Limitations

- Differences between fate-group GRN configurations are subtle; detection depends on the permutation-test clone-membership split and the small group sizes (55 vs 50 cells)
- Fate groups have asymmetric clone counts (9 enriched vs 43 depleted clones), reflecting heterogeneous clone contributions among converted cells
- CellTag barcode diversity is limited by bottlenecks during synthesis and cloning, so plasmid-library allowlisting is required before clone calling
- The finding that iEPs resemble post-injury biliary epithelial cells is attributed to a parallel study (Kong et al., 2022) and is not established here

## Evidence

- [results] Comparing GRN configurations of day-4 reprogrammed-destined versus dead-end-destined clones reveals subtle differences, with Klf6, Tbx5, Tfapb2, and Foxs1 more highly connected in failing cells, versus Fos, Klf5, and Junb more highly connected in cells destined to attain full iEP identity.: [evidence span withheld — anchor status: not_found]
