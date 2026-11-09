---
name: nonlinear-manifold-network-alignment
description: Use when two condition-specific scGRNs (e.g., mock vs morphine-treated
  neurons) have been denoised by CP tensor decomposition and you need a common embedding
  to quantify gene-wise regulatory differences.
when_to_use_negative:
- Input networks have not been denoised by CP tensor decomposition — raw subsampled
  pcNet networks are too noisy for meaningful alignment
- The two samples are highly dissimilar populations (e.g., different cell types) —
  the article notes scTenifoldNet is more suitable for comparing highly similar samples
  such as two populations of cells of the same type
- The goal is de novo network construction from expression data rather than comparison
  of already-built networks
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scTenifoldNet
  role: R package whose manifoldAlignment function performs non-linear alignment of
    the two denoised networks (step 5 of the pipeline); installed from CRAN or GitHub
  repo: https://github.com/cailab-tamu/scTenifoldNet
- name: R
  role: Runtime environment; scTenifoldNet is installed with install.packages('scTenifoldNet')
- name: igraph
  role: Optional conversion of the sparse tensorNetworks (dgCMatrix) to graph objects
    for downstream topological checks around the alignment step
  repo: https://igraph.org/r/
- name: scTenifoldpy
  role: Python implementation of the scTenifold suite including scTenifoldNet for
    cross-condition GRN comparison
  repo: https://github.com/qwerty239qwe/scTenifoldpy
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2020.100139
    title: 'scTenifoldNet: A Machine Learning Workflow for Constructing and Comparing
      Transcriptome-wide Gene Regulatory Networks from Single-Cell Data'
  source_content_hash: sha256:da00ec7dfcacf5bfde2e5aaef6d751634dfc99827e7f2715c833ae4130065d90
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cailab-tamu/scTenifoldNet
  repo_url: https://github.com/cailab-tamu/scTenifoldNet
  tools:
  - R
  - igraph
  - scTenifoldNet
  - scTenifoldpy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# nonlinear-manifold-network-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns two denoised single-cell gene regulatory networks (scGRNs) into a shared low-dimensional latent space using nonlinear manifold alignment, so that per-gene coordinate shifts between conditions can be measured for differential regulation testing.

## When to use

Use when two condition-specific scGRNs (e.g., mock vs morphine-treated neurons) have been denoised by CP tensor decomposition and you need a common embedding to quantify gene-wise regulatory differences.

## When NOT to use

- Input networks have not been denoised by CP tensor decomposition — raw subsampled pcNet networks are too noisy for meaningful alignment
- The two samples are highly dissimilar populations (e.g., different cell types) — the article notes scTenifoldNet is more suitable for comparing highly similar samples such as two populations of cells of the same type
- The goal is de novo network construction from expression data rather than comparison of already-built networks

## Inputs

- Denoised scGRN for sample X (sparse matrix, dgCMatrix) from tensorDecomposition
- Denoised scGRN for sample Y (sparse matrix, dgCMatrix) from tensorDecomposition
- Manifold alignment dimensionality parameter d (default 30)

## Outputs

- manifoldAlignment data frame of low-dimensional features with 2 × n genes rows and d columns (default d = 30)
- Per-gene distances between condition-specific coordinates, consumed by dRegulation as distance, Z, FC, p.value, and p.adj (Benjamini-Hochberg FDR) columns

## How to apply

After constructing per-condition networks with pcNet/makeNetworks and denoising them with CANDECOMP/PARAFAC tensorDecomposition (td_K = 3), run scTenifoldNet's manifoldAlignment on the two sparse dgCMatrix tensor networks to produce low-dimensional features for 2 × n genes (default d = 30 dimensions). The alignment embeds corresponding genes of networks X and Y into one shared space so that a gene whose regulatory neighborhoods changed occupies different coordinates in each condition. Feed the resulting alignment data frame into dRegulation, which computes each gene's Euclidean distance between condition-specific coordinates, applies a Box-Cox transformation, and derives chi-square(1) p-values with Benjamini-Hochberg FDR correction. The step is step 5 of the six-step scTenifoldNet() pipeline and can be run independently since all pipeline functions are exported and documented. With real data (e.g., the Avey et al. mock vs morphine nucleus accumbens comparison), this alignment underpins identification of ~56 differentially regulated genes enriched for opioid signaling, GPCR signaling, and calcium-signaling terms.

## Related tools

- **scTenifoldNet** (R package whose manifoldAlignment function performs non-linear alignment of the two denoised networks (step 5 of the pipeline); installed from CRAN or GitHub) — https://github.com/cailab-tamu/scTenifoldNet
- **R** (Runtime environment; scTenifoldNet is installed with install.packages('scTenifoldNet'))
- **igraph** (Optional conversion of the sparse tensorNetworks (dgCMatrix) to graph objects for downstream topological checks around the alignment step) — https://igraph.org/r/
- **scTenifoldpy** (Python implementation of the scTenifold suite including scTenifoldNet for cross-condition GRN comparison) — https://github.com/qwerty239qwe/scTenifoldpy

## Examples

```
output <- scTenifoldNet(X = X, Y = Y, nc_nNet = 10, nc_nCells = 500, td_K = 3, qc_minLibSize = 30); dim(output$manifoldAlignment)
```

## Evaluation signals

- The manifoldAlignment data frame has exactly 2 × n genes rows and d columns (default d = 30), matching the documented output schema
- H0 sanity check: aligning a network to itself (X vs X) yields no gene with FDR < 0.1 after dRegulation (e.g., top gene p.adj = 0.6411916 in the README example)
- Positive control: on the six-gene swap perturbation (genes 10, 2, 3 replaced by 50, 11, 5), exactly the six swapped genes (ng2, ng50, ng11, ng3, ng10, ng5) rank top with p.adj < 0.1
- QQ-plot of the dRegulation FC statistic against chi-square(1) quantiles should track the reference line under H0, with significant genes (FDR < 0.1) highlighted in red only under HA
- On the Avey et al. morphine dataset, ranked genes should reproduce ~56 significant genes (FDR < 0.05) enriched for opioid signaling, GPCR signaling, reduction of cytosolic calcium levels, and morphine addiction

## Limitations

- Constructing scGRN from scRNA-seq data consisting of cells in many different states is challenging at present, which propagates into the alignment input
- It is difficult to control for technical noise in scRNA-seq data, affecting the reliability of the aligned embeddings
- scTenifoldNet is more suitable for comparing highly similar samples, such as two populations of cells of the same type, restricting applicability of the alignment
- Causal factors and their target genes remain unknown in many biological systems, limiting ground-truth validation of alignment-derived differential regulation results

## Evidence
