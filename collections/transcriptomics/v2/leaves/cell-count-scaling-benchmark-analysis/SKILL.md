---
name: cell-count-scaling-benchmark-analysis
description: Use when you must show how reconstruction accuracy and sensitivity of
  a scGRN method depend on the number of input cells, and ground-truth regulatory
  edges (e.g., SERGIO-simulated 100-gene modules) are available for scoring.
when_to_use_negative:
- Analyzing real scRNA-seq data with no ground-truth regulatory edges available —
  accuracy/recall against SERGIO truth cannot be computed
- The goal is detecting differential regulation between two biological conditions
  rather than benchmarking network construction (use the full scTenifoldNet pipeline
  with tensorDecomposition and manifoldAlignment instead)
- Input matrices are already normalized/processed rather than raw counts — scTenifoldNet
  requires unnormalized input for its main workflow
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: scTenifoldNet
  role: Provides pcNet and makeNetworks for principal-component regression scGRN construction
    from subsampled cells at each cell-number setting
  repo: github.com/cailab-tamu/scTenifoldNet
- name: R
  role: Language in which scTenifoldNet, the baseline reconstructions, scoring, and
    Figure 2A plotting are implemented
- name: GENIE3
  role: Baseline GRN construction method used as the comparison point against PC regression
    in the cell-count scaling benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2020.100139
    title: 'scTenifoldNet: A Machine Learning Workflow for Constructing and Comparing
      Transcriptome-wide Gene Regulatory Networks from Single-Cell Data'
  source_content_hash: sha256:65b7e139015d26bec509dc62ade3247d0135571c866c22b318cf5045dfc36656
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cailab-tamu/scTenifoldNet
  tools:
  - GENIE3
  - R
  - scTenifoldNet
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# cell-count-scaling-benchmark-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks GRN construction algorithms (PC regression via scTenifoldNet vs Spearman correlation, mutual information, and GENIE3) across input-cell-number settings by scoring reconstructed scGRNs against SERGIO ground-truth regulatory edges, demonstrating that PC regression yields superior accuracy (specificity) and recall (sensitivity).

## When to use

Use when you must show how reconstruction accuracy and sensitivity of a scGRN method depend on the number of input cells, and ground-truth regulatory edges (e.g., SERGIO-simulated 100-gene modules) are available for scoring.

## When NOT to use

- Analyzing real scRNA-seq data with no ground-truth regulatory edges available — accuracy/recall against SERGIO truth cannot be computed
- The goal is detecting differential regulation between two biological conditions rather than benchmarking network construction (use the full scTenifoldNet pipeline with tensorDecomposition and manifoldAlignment instead)
- Input matrices are already normalized/processed rather than raw counts — scTenifoldNet requires unnormalized input for its main workflow

## Inputs

- SERGIO-simulated scRNA-seq raw count matrices (100 genes in two regulatory modules; genes as rows, cells as columns; cell counts varied from 10 to 3,000)
- SERGIO ground-truth regulatory edge list defining true gene regulatory interactions
- Reconstructed scGRNs (weighted adjacency matrices) from PC regression (scTenifoldNet pcNet/makeNetworks), Spearman correlation, mutual information, and GENIE3 at each cell-number setting

## Outputs

- Per-method, per-cell-number accuracy (specificity) and recall (sensitivity) values scored against SERGIO ground-truth edges
- Figure 2A curves of accuracy and recall versus number of cells for all four methods (saved as figure_2a_accuracy_recall.png)

## How to apply

Generate SERGIO-simulated scRNA-seq raw count matrices with 100 genes in two regulatory modules, varying cell counts from 10 to 3,000 (genes as rows, cells as columns). At each cell-number setting, reconstruct scGRNs with scTenifoldNet's PC regression (pcNet/makeNetworks functions on subsampled cells) and with the three baselines: Spearman correlation, mutual information, and GENIE3. Score each reconstructed network against the SERGIO ground-truth regulatory edges, computing accuracy (specificity) and recall (sensitivity) per method per setting. Plot accuracy and recall versus number of cells for all four methods to reproduce the Figure 2A curves, then verify that the PC regression curves lie above the SCC, MI, and GENIE3 curves for both metrics across the whole cell-number range, consistent with the finding that PC regression outperforms the other GRN construction algorithms in specificity and sensitivity.

## Related tools

- **scTenifoldNet** (Provides pcNet and makeNetworks for principal-component regression scGRN construction from subsampled cells at each cell-number setting) — github.com/cailab-tamu/scTenifoldNet
- **R** (Language in which scTenifoldNet, the baseline reconstructions, scoring, and Figure 2A plotting are implemented)
- **GENIE3** (Baseline GRN construction method used as the comparison point against PC regression in the cell-count scaling benchmark)

## Examples

```
install.packages("scTenifoldNet"); output <- scTenifoldNet(X = X, Y = X, nc_nNet = 10, nc_nCells = 500, td_K = 3, qc_minLibSize = 30)
```

## Evaluation signals

- The PC regression accuracy and recall curves lie above the Spearman, mutual information, and GENIE3 curves for both metrics across the full 10–3,000 cell range
- Accuracy and recall are computed per method per cell-number setting against the SERGIO ground-truth regulatory edges (specificity = accuracy, sensitivity = recall)
- Figure 2A is reproduced and saved as figure_2a_accuracy_recall.png with accuracy and recall versus number of cells for all four methods
- The superiority of PC regression holds across a wide range of cell numbers, not only at high cell counts, consistent with the reported minimum-cells requirement result

## Limitations

- Constructing scGRNs from scRNA-seq data containing cells in many different states is challenging at present
- Regulatory relationships between genes are difficult to establish from scRNA-seq data even though the data may theoretically capture a complete picture of the regulatory gene landscape
- Benchmark validity depends on SERGIO-simulated ground truth; causal factors and their target genes remain unknown in many biological systems studied, limiting real-data validation

## Evidence

- [discussion] PC regression outperforms other GRN construction algorithms: "PC regression significantly outperforms the other GRN construction algorithms in all aspects of methodology metrics, including specificity, sensitivity, computational efficiency, and the required"
- [intro] Challenge motivating benchmarking: "Constructing single-cell GRNs (scGRNs) using data from scRNA-seq and then effectively comparing constructed scGRNs presents significant analytical challenges."
- [methods] pcNet signature: "pcNet(X, nComp, scaleScores, symmetric, q, priorNetwork, verbose, nCores, useRcpp)"
- [readme] makeNetworks description: "Constructs gene regulatory networks from subsampled cells using principal component regression"
- [readme] Raw counts input format: "The required input is a **raw counts matrix** with genes as rows and cells (barcodes) as columns."
