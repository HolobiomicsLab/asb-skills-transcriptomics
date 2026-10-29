---
name: single-cell-clone-calling-from-lineage-barcode-data
description: Use when single-cell scRNA-seq data carry integrated lineage barcodes
  (e.g. CellTag lentiviral captures from 10x Chromium runs) and you must group cells
  into clones for state-fate or clonal GRN analyses.
when_to_use_negative:
- The dataset contains no lineage barcodes (plain scRNA-seq without CellTag or similar
  lineage capture) — clone calling is impossible and cell clustering should be used
  instead
- The goal is cell-type classification or fate-transition scoring of transcriptomes
  only — Capybara is the appropriate tool, not barcode-based clone calling
- The barcode library has not been allowlisted against the plasmid library — calling
  clones from unallowed barcodes will inflate clone counts
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0102
tools:
- name: newCloneCalling
  role: Runs allowlisting of CellTag plasmid barcodes and clone calling on integrated
    barcode data to recover clonal lineages
  repo: https://github.com/morris-lab/newCloneCalling
- name: Cell Ranger 6.0.1
  role: Processes the 10x Chromium scRNA-seq data to generate the digital gene expression
    matrix that barcode data are integrated with
- name: Capybara
  role: Classifies cells along the reprogramming time course so clone calls can be
    linked to day-4 states and iEP conversion outcomes
  repo: https://github.com/morris-lab/Capybara
- name: CellOracle
  role: Consumes clone-derived fate groups to build sample-specific GRN/Links objects
    and compute igraph-based TF degree centrality
  repo: https://github.com/morris-lab/CellOracle
- name: igraph
  role: R library used for network scoring (TF degree centrality) on the GRNs built
    from clone-defined cell groups
  repo: https://github.com/igraph/rigraph
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:0f6b49f599e37a56a8b2e72ee85934f12bbc444e20461e684eda83fdc993d8d9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/morris-lab/CellOracle
  repo_url: https://github.com/morris-lab/newCloneCalling
  tools:
  - Capybara
  - Cell Ranger
  - Cell Ranger 6.0.1
  - CellOracle
  - igraph
  - newCloneCalling
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-clone-calling-from-lineage-barcode-data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Recover clonal lineages from single-cell lineage-tracing datasets by allowlisting lineage barcodes (e.g. CellTag lentiviral barcodes) and calling clones from integrated barcode data, producing clone labels per cell that link early cell states to fate outcomes.

## When to use

Use when single-cell scRNA-seq data carry integrated lineage barcodes (e.g. CellTag lentiviral captures from 10x Chromium runs) and you must group cells into clones for state-fate or clonal GRN analyses.

## When NOT to use

- The dataset contains no lineage barcodes (plain scRNA-seq without CellTag or similar lineage capture) — clone calling is impossible and cell clustering should be used instead
- The goal is cell-type classification or fate-transition scoring of transcriptomes only — Capybara is the appropriate tool, not barcode-based clone calling
- The barcode library has not been allowlisted against the plasmid library — calling clones from unallowed barcodes will inflate clone counts

## Inputs

- Digital gene expression matrix from Cell Ranger v6.0.1 for a 10x Chromium scRNA-seq run
- Integrated lineage barcode data (CellTag lentiviral barcode captures per cell)
- Plasmid lineage-barcode library sequencing data for allowlisting
- Cell metadata with reprogramming time-course state labels (e.g. Capybara day-4 state and iEP outcome assignments)

## Outputs

- Per-cell clone assignments (1,158 called clones in the reference reprogramming dataset)
- Fate-grouped cell/clone sets (e.g. 55 cells in 9 iEP-enriched clones vs. 50 cells in 43 iEP-depleted clones) for downstream GRN comparison

## How to apply

Start from the digital gene expression matrix generated with Cell Ranger v6.0.1 for the 10x Chromium scRNA-seq run (24,799 cells in the reference reprogramming dataset). First allowlist CellTag barcodes by sequencing the plasmid library, since real barcode diversity is bottlenecked relative to theoretical diversity (65,536 for original tags, ~68 billion for 18N libraries), then run the newCloneCalling pipeline (github.com/morris-lab/newCloneCalling) on the integrated barcode data to call clones (1,158 clones in the reference run). Validate clone counts before downstream use, then pass per-cell clone labels to fate-grouping (e.g. a randomized permutation test of clone membership split day-4 cells into 55 cells/9 iEP-enriched clones versus 50 cells/43 iEP-depleted clones) and to sample-specific CellOracle GRN construction with igraph-based TF degree centrality scoring. Correct application means the recovered clone count matches expectations and downstream fate comparisons (e.g. Figure 2H degree-centrality separation) reproduce.

## Related tools

- **newCloneCalling** (Runs allowlisting of CellTag plasmid barcodes and clone calling on integrated barcode data to recover clonal lineages) — https://github.com/morris-lab/newCloneCalling
- **Cell Ranger 6.0.1** (Processes the 10x Chromium scRNA-seq data to generate the digital gene expression matrix that barcode data are integrated with)
- **Capybara** (Classifies cells along the reprogramming time course so clone calls can be linked to day-4 states and iEP conversion outcomes) — https://github.com/morris-lab/Capybara
- **CellOracle** (Consumes clone-derived fate groups to build sample-specific GRN/Links objects and compute igraph-based TF degree centrality) — https://github.com/morris-lab/CellOracle
- **igraph** (R library used for network scoring (TF degree centrality) on the GRNs built from clone-defined cell groups) — https://github.com/igraph/rigraph

## Examples

```
git clone https://github.com/morris-lab/newCloneCalling && Rscript newCloneCalling/run_clone_calling.R --celltags celltag_barcodes.tsv --allowlist plasmid_allowlist.tsv
```

## Evaluation signals

- Recovered clone count matches the expected value (1,158 clones for the 24,799-cell reprogramming dataset)
- Fate-group split reproduces the reported sizes: 9 iEP-enriched clones (55 cells) and 43 iEP-depleted clones (50 cells) at day 4
- Sister cells (same clone) map to the same reprogramming trajectory, consistent with conversion outcome being established shortly after TF overexpression
- Downstream Figure 2H plot shows the reported TF degree-centrality separation between reprogrammed-destined and dead-end-destined clones (Fos/Klf5/Junb higher in iEP-destined; Klf6/Tbx5/Tfapb2/Foxs1 higher in failing cells)
- All called barcodes belong to the plasmid-library allowlist (no unallowed barcodes contributing clones)

## Limitations

- Real CellTag barcode diversity in the plasmid library is limited by bottlenecks during synthesis and cloning, so allowlisting against the plasmid library is mandatory before clone calling
- iEP-enriched fate groups are small (55 cells/9 clones vs. 50 cells/43 clones), so GRN comparisons rest on modest cell numbers and subtle network differences
- newCloneCalling is specific to CellTag-multi-compatible barcode designs; it is not a general clone caller for arbitrary lineage-tracing chemistries

## Evidence

- [readme] Allowlisting via plasmid library: [evidence span withheld — anchor status: external_doc]
