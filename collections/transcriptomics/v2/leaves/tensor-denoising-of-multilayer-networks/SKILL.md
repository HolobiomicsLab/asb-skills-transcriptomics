---
name: tensor-denoising-of-multilayer-networks
description: Use when you have constructed multiple candidate scGRNs per condition
  (e.g., 10 networks from 500-cell subsamples via pcNet/makeNetworks) and need a single
  denoised network per condition before aligning conditions or testing differential
  regulation.
when_to_use_negative:
- You have only a single pre-built GRN per condition (no multilayer/subsampled network
  stack) — the CP tensor approximation requires a tensor of multiple network layers
  to denoise
- You want to compare highly dissimilar samples rather than two populations of cells
  of the same type; scTenifoldNet as a whole is designed for highly similar samples
- Input is still raw single-cell count data — QC (scQC) and normalization (cpmNormalization)
  and network construction (makeNetworks) must precede tensor denoising
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scTenifoldNet
  role: R package providing the tensorDecomposition function that applies CP tensor
    decomposition for network denoising within the scTenifoldNet pipeline
  repo: github.com/cailab-tamu/scTenifoldNet
- name: scTenifoldpy
  role: Python implementation of the scTenifold suite including the cross-condition
    GRN comparison workflow with tensor denoising
  repo: https://github.com/qwerty239qwe/scTenifoldpy
- name: igraph
  role: Optional downstream conversion of the denoised sparse adjacency matrices (dgCMatrix)
    to igraph objects for topological analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2020.100139
    title: 'scTenifoldNet: A Machine Learning Workflow for Constructing and Comparing
      Transcriptome-wide Gene Regulatory Networks from Single-Cell Data'
  source_content_hash: sha256:5afd1a2120d2c00b81a0e8266c37666aacb2cfcabb929e15ddf5eb1e7d95a253
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
  - scTenifoldpy
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# tensor-denoising-of-multilayer-networks

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Denoises the multilayer set of single-cell gene regulatory networks (scGRNs) built from subsampled cells using CANDECOMP/PARAFAC (CP) low-rank tensor decomposition, which smooths edge weights across networks so downstream manifold alignment and differential regulation testing operate on a single robust network per condition.

## When to use

Use when you have constructed multiple candidate scGRNs per condition (e.g., 10 networks from 500-cell subsamples via pcNet/makeNetworks) and need a single denoised network per condition before aligning conditions or testing differential regulation.

## When NOT to use

- You have only a single pre-built GRN per condition (no multilayer/subsampled network stack) — the CP tensor approximation requires a tensor of multiple network layers to denoise
- You want to compare highly dissimilar samples rather than two populations of cells of the same type; scTenifoldNet as a whole is designed for highly similar samples
- Input is still raw single-cell count data — QC (scQC) and normalization (cpmNormalization) and network construction (makeNetworks) must precede tensor denoising

## Inputs

- Multilayer stack of gene regulatory networks per condition (gene-by-gene adjacency matrices from makeNetworks/pcNet applied to randomly subsampled cells)
- td_K parameter (rank of the CP tensor approximation; e.g., td_K = 3 in the package example settings)

## Outputs

- Weight-averaged denoised gene regulatory networks after CP tensor decomposition: tensorNetworks$X and tensorNetworks$Y (sparse dgCMatrix adjacency matrices, one per condition)
- Denoised networks passed to manifoldAlignment for cross-condition network comparison

## How to apply

After makeNetworks has produced a collection of gene-by-gene adjacency networks from randomly subsampled cells (e.g., nc_nNet = 10 networks of nc_nCells = 500 cells each), apply the tensorDecomposition function of scTenifoldNet to the multilayer network stack. The function performs CANDECOMP/PARAFAC (CP) tensor decomposition, a low-rank tensor approximation, to smooth edge weights across all networks in the multilayer scGRN; in the morphine-vs-mock workflow (Avey et al. data) the package example setting td_K = 3 is used. The output is a weight-averaged denoised network per condition, returned as sparse dgCMatrix objects (tensorNetworks$X and tensorNetworks$Y), which are then aligned by manifoldAlignment and subjected to dRegulation testing. Denoising is essential because scRNA-seq-derived networks are noisy and sparse; the tensor approximation suppresses spurious edges that appear in individual subsampled networks. The denoised sparse matrices can be converted to igraph objects for downstream topological inspection.

## Related tools

- **scTenifoldNet** (R package providing the tensorDecomposition function that applies CP tensor decomposition for network denoising within the scTenifoldNet pipeline) — github.com/cailab-tamu/scTenifoldNet
- **scTenifoldpy** (Python implementation of the scTenifold suite including the cross-condition GRN comparison workflow with tensor denoising) — https://github.com/qwerty239qwe/scTenifoldpy
- **igraph** (Optional downstream conversion of the denoised sparse adjacency matrices (dgCMatrix) to igraph objects for topological analysis)

## Examples

```
output <- scTenifoldNet(X = mockCounts, Y = morphineCounts, nc_nNet = 10, nc_nCells = 500, td_K = 3, qc_minLibSize = 30); netX <- output$tensorNetworks$X
```

## Evaluation signals

- The output tensorNetworks element contains exactly two sparse dgCMatrix entries, X and Y, matching the two input conditions and gene dimensionality
- Denoised edge weights are consistent across the multilayer stack — edges appearing in only one subsampled network are suppressed relative to edges recurrent across networks (tensor denoising smooths edge weights across all networks)
- Running the full pipeline under H0 (same matrix for both conditions) yields no genes with FDR < 0.1, confirming denoising does not introduce spurious differential regulation
- Under HA (perturbed matrices), the known perturbed genes rank at the top of diffRegulation with significant FDR (e.g., ng2/ng50 p.adj ≈ 0.024 in the README example), indicating the denoised networks preserve true regulatory signal
- Runtime/memory should be consistent with benchmarks in the README (e.g., ~3 h for 5,000 cells × 5,000 genes), with tensor decomposition only a fraction of the total pipeline time dominated by network construction

## Limitations

- Denoising is only as good as the underlying subsampled networks; regulatory relationships between genes are difficult to establish from scRNA-seq data, even though the data may theoretically capture a complete picture of the regulatory gene landscape
- Technical noise in scRNA-seq data is difficult to control for, so tensor denoising cannot fully remove noise originating from upstream library preparation and sequencing
- Comparing cells in many different states is challenging; scTenifoldNet (and its denoising step) is more suitable for comparing highly similar samples, such as two populations of cells of the same type
- The choice of the CP approximation rank (td_K, e.g., 3 in package examples) controls the denoising strength; inappropriate ranks may over- or under-smooth edge weights

## Evidence
