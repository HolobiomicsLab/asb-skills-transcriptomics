---
name: network-inference-method-comparison
description: Use when choosing a GRN construction method for scRNA-seq data, or when
  reproducing benchmark curves of accuracy/recall versus cell count against known
  regulatory edges.
when_to_use_negative:
- The input is real scRNA-seq data with no known ground-truth regulatory edges — accuracy/recall
  cannot be computed, so only indirect quality assessments apply
- The goal is cross-condition differential regulation analysis of two similar cell
  populations rather than method benchmarking — use the full scTenifoldNet workflow
  (tensorDecomposition, manifoldAlignment, dRegulation) instead
- Data are already normalized rather than raw counts — scTenifoldNet expects unnormalized
  input for its main pipeline
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0102
tools:
- name: scTenifoldNet
  role: Provides pcNet/makeNetworks for principal-component regression GRN construction
    on subsampled cells (the method under test)
  repo: github.com/cailab-tamu/scTenifoldNet
- name: R
  role: Implementation environment for scTenifoldNet and for scoring/plotting the
    benchmark curves
- name: igraph
  role: Optional downstream topological analysis of the reconstructed networks as
    graph objects
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2020.100139
    title: 'scTenifoldNet: A Machine Learning Workflow for Constructing and Comparing
      Transcriptome-wide Gene Regulatory Networks from Single-Cell Data'
  source_content_hash: sha256:ce8a0b38f1749f13aa7b1f070aadcb27e68eb1eee9c85b3cf65e3c3cac4901da
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cailab-tamu/scTenifoldNet
  tools:
  - R
  - igraph
  - scTenifoldNet
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# network-inference-method-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark gene regulatory network (GRN) construction algorithms — PC regression (via scTenifoldNet's pcNet/makeNetworks), Spearman correlation, mutual information, and GENIE3 — against ground-truth edges in simulated scRNA-seq data, scoring accuracy (specificity) and recall (sensitivity) as a function of cell number.

## When to use

Use when choosing a GRN construction method for scRNA-seq data, or when reproducing benchmark curves of accuracy/recall versus cell count against known regulatory edges.

## When NOT to use

- The input is real scRNA-seq data with no known ground-truth regulatory edges — accuracy/recall cannot be computed, so only indirect quality assessments apply
- The goal is cross-condition differential regulation analysis of two similar cell populations rather than method benchmarking — use the full scTenifoldNet workflow (tensorDecomposition, manifoldAlignment, dRegulation) instead
- Data are already normalized rather than raw counts — scTenifoldNet expects unnormalized input for its main pipeline

## Inputs

- SERGIO-simulated scRNA-seq raw count matrix (genes as rows, cells as columns, 100 genes, 10–3,000 cells)
- Ground-truth regulatory edges from the simulated regulatory modules
- Reconstructed adjacency/weight networks from PC regression, SCC, MI, and GENIE3

## Outputs

- Per-method, per-cell-count accuracy (specificity) and recall (sensitivity) scores against ground-truth edges
- Accuracy and recall versus number-of-cells curves (Figure 2A reproduction, e.g. figure_2a_accuracy_recall.png)

## How to apply

Generate SERGIO-simulated raw count matrices (genes as rows, cells as columns) with ~100 genes in two regulatory modules, varying cell counts from 10 to 3,000. At each cell count, reconstruct scGRNs with PC regression using scTenifoldNet's pcNet/makeNetworks on subsampled cells, and in parallel with three baselines: Spearman correlation (SCC), mutual information (MI), and GENIE3. Score each reconstructed network against the SERGIO ground-truth regulatory edges, computing accuracy (specificity) and recall (sensitivity) per method per cell-number setting. Plot accuracy and recall versus cell number for all four methods to reproduce the Figure 2A curves; PC regression should lie above SCC, MI, and GENIE3 on both metrics, reflecting its reported superiority in specificity, sensitivity, and computational efficiency.

## Related tools

- **scTenifoldNet** (Provides pcNet/makeNetworks for principal-component regression GRN construction on subsampled cells (the method under test)) — github.com/cailab-tamu/scTenifoldNet
- **R** (Implementation environment for scTenifoldNet and for scoring/plotting the benchmark curves)
- **igraph** (Optional downstream topological analysis of the reconstructed networks as graph objects)

## Examples

```
Rscript -e 'network <- pcNet(countsMatrix, nComp = 3, scaleScores = TRUE, symmetric = TRUE, verbose = TRUE)'
```

## Evaluation signals

- PC regression accuracy and recall curves lie above the Spearman, mutual information, and GENIE3 curves across the full 10–3,000 cell range
- Scores are computed against the SERGIO ground-truth regulatory edges, not against inferred targets
- The x-axis spans the full 10–3,000 cell range used in the simulation design
- Networks are built from raw counts with genes as rows and cells as columns, matching scTenifoldNet's expected input orientation

## Limitations

- Benchmark results depend on simulated data (SERGIO); causal factors and their target genes remain unknown in many biological systems, limiting ground-truth validation with real data
- Constructing scGRNs from data containing cells in many different states is challenging, and technical noise in scRNA-seq is difficult to control
- Comparison is restricted to the four tested algorithms and simulated 100-gene two-module systems, so conclusions may not transfer to all real GRN settings

## Evidence
