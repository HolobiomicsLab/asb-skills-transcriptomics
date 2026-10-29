---
name: ground-truth-regulatory-relationship-recovery-evaluation
description: Use when validating GRN inference or denoising methods against a simulated
  scRNA-seq dataset with known regulatory relationships, e.g., quantifying recall
  gains from CP tensor denoising of multilayer scGRNs.
when_to_use_negative:
- Input is real scRNA-seq data with no known ground-truth regulatory relationships
  — recall cannot be computed and other validation (e.g., functional enrichment of
  ranked genes) is needed
- The goal is differential regulation testing between two conditions (manifoldAlignment/dRegulation),
  not assessing recovery of simulated edges
- The input is already a denoised network with no matched set of individual (pre-denoising)
  networks to compare against
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scTenifoldNet
  role: Provides makeNetworks/pcNet for inferring networks from 500-cell subsamples
    and tensorDecomposition (CP, td_K = 3) for the denoised consensus network whose
    recall is evaluated
  repo: github.com/cailab-tamu/scTenifoldNet
- name: RSpectra
  role: Imported dependency supporting the spectral computations in PC regression
    network construction
- name: igraph
  role: Optional conversion of the denoised dgCMatrix networks into graph objects
    for downstream topological inspection of recovered edges
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2020.100139
    title: 'scTenifoldNet: A Machine Learning Workflow for Constructing and Comparing
      Transcriptome-wide Gene Regulatory Networks from Single-Cell Data'
  source_content_hash: sha256:5ef03d5fe204db4e6ddd46ea901fad4a5a03648d1873f349c96ca89151a27ff0
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
  - RSpectra
  - Rcpp
  - igraph
  - scTenifoldNet
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# ground-truth-regulatory-relationship-recovery-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how faithfully inferred single-cell GRNs (scGRNs) recover known regulatory relationships from a simulated scRNA-seq ground truth, by computing recall of ground-truth edges before and after CP tensor-denoising of multilayer networks. It provides the validation metric (recall improvement, ~25%) used to justify tensor denoising in the scTenifoldNet workflow.

## When to use

Use when validating GRN inference or denoising methods against a simulated scRNA-seq dataset with known regulatory relationships, e.g., quantifying recall gains from CP tensor denoising of multilayer scGRNs.

## When NOT to use

- Input is real scRNA-seq data with no known ground-truth regulatory relationships — recall cannot be computed and other validation (e.g., functional enrichment of ranked genes) is needed
- The goal is differential regulation testing between two conditions (manifoldAlignment/dRegulation), not assessing recovery of simulated edges
- The input is already a denoised network with no matched set of individual (pre-denoising) networks to compare against

## Inputs

- Simulated raw count matrix (cells x genes) from a negative binomial distribution (rnbinom, size = 20, prob = 0.98, ~67% zeros) with mt- prefixed mitochondrial genes
- Ground-truth simulated regulatory relationships (which genes carry which profiles)
- Individual inferred networks from repeated subsampling (nc_nNet = 10, nc_nCells = 500)
- CP-denoised consensus network from tensorDecomposition at rank K = 3 (td_K = 3)

## Outputs

- recall_improvement.csv — tabulated recall before vs. after denoising with percentage improvement
- tensor_networks.rds — denoised networks stored as RDS
- Per-network and consensus recall rates of ground-truth regulatory relationships

## How to apply

Simulate a raw count matrix (e.g., 2,000 cells x 100 genes from rnbinom with size = 20, prob = 0.98, ~67% zeros, ten mt- prefixed mitochondrial genes) and record the ground-truth simulated regulatory relationships (genes 10, 2, and 3 carrying the profiles of genes 50, 11, and 5). Draw 10 random subsamples of 500 cells (nc_nNet = 10, nc_nCells = 500) and infer a GRN from each via principal component regression (scTenifoldNet's makeNetworks/pcNet), forming a multilayer scGRN. Denoise the stack of 10 networks with CANDECOMP/PARAFAC tensor decomposition (tensorDecomposition, td_K = 3), obtaining weight-averaged tensorNetworks as sparse dgCMatrix matrices. Compute recall of the ground-truth relationships separately for the individual pre-denoising networks and the CP-denoised consensus network, tabulate the percentage improvement, and save recall_improvement.csv and tensor_networks.rds.

## Related tools

- **scTenifoldNet** (Provides makeNetworks/pcNet for inferring networks from 500-cell subsamples and tensorDecomposition (CP, td_K = 3) for the denoised consensus network whose recall is evaluated) — github.com/cailab-tamu/scTenifoldNet
- **RSpectra** (Imported dependency supporting the spectral computations in PC regression network construction)
- **igraph** (Optional conversion of the denoised dgCMatrix networks into graph objects for downstream topological inspection of recovered edges)

## Examples

```
outputH0 <- scTenifoldNet(X = X, Y = X, nc_nNet = 10, nc_nCells = 500, td_K = 3, qc_minLibSize = 30)
```

## Evaluation signals

- Recall improvement of the CP-denoised multilayer scGRN over the individual networks approximates the ~25% gain reported for Figure 2B; accept a stochastic tolerance band around 25% (e.g., 15-35%)
- The known perturbation relationships (gene 10 carrying gene 50's profile, gene 2 carrying gene 11's, gene 3 carrying gene 5's) are recovered among the top hits
- Under H0 (same matrix in both conditions), no genes reach significance (FDR < 0.1), confirming the evaluation is not picking up spurious edges
- Denoised tensorNetworks are returned as sparse dgCMatrix matrices (one per condition, X and Y), suitable for graph conversion
- Recall comparison table is saved as recall_improvement.csv and denoised networks as tensor_networks.rds

## Limitations

- Causal factors and their target genes remain unknown in many biological systems, limiting ground-truth validation beyond simulation
- It is difficult to control for technical noise in scRNA-seq data, so recall on simulated data may not transfer to real datasets
- scTenifoldNet-based multilayer construction is more suitable for comparing highly similar samples, restricting generalization of the simulated recall benchmark

## Evidence

- [other] Tensor denoising improves the recall rate of recovered regulatory relationships between genes by 25% relative to the ground-truth simulated scGRN: [evidence span withheld — anchor status: not_found]
