---
name: coexpression-prior-regularized-dynamics-fitting
description: Use when you need per-cell RNA velocity vectors paired with an interpretable
  gene regulatory network from single-cell transcriptomics, especially when spliced/unspliced-only
  methods give orthogonal or inconsistent directions relative to a known biological
  transition axis.
when_to_use_negative:
- When you only need fast standard velocity without a mechanistic GRN and spliced/unspliced
  counts are already adequate for scVelo-style fitting
- When no gene-gene coexpression prior, epigenetic (ATAC-seq) mask, or cluster structure
  is available to initialize the priors and manifold constraint
- When the dataset is too small to support per-cluster EM fitting of a full 15497-gene
  interaction matrix
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scKINETICS
  role: 'Core dynamical model: EM fitting of per-cell velocities and gene regulatory
    network using coexpression, epigenetic, and manifold priors'
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Accompanying notebook demonstrating input file generation, GRN construction,
    EM estimation, visualizations, and TF ablation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:c8c80ec24557591ccf701eb9215fc201aac0a5cbe534668b8f8b99ba8a56d0ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# coexpression-prior-regularized-dynamics-fitting

## Summary

Fit scKINETICS, a dynamical model of gene expression change that simultaneously learns per-cell transcriptional velocities and a governing gene regulatory network via expectation-maximization, regularized by gene-gene coexpression priors and phenotypic manifold constraints. It yields interpretable, mechanistic velocity vectors where standard approaches (scVelo, VeloVAE, UniTVelo) often fail.

## When to use

Use when you need per-cell RNA velocity vectors paired with an interpretable gene regulatory network from single-cell transcriptomics, especially when spliced/unspliced-only methods give orthogonal or inconsistent directions relative to a known biological transition axis.

## When NOT to use

- When you only need fast standard velocity without a mechanistic GRN and spliced/unspliced counts are already adequate for scVelo-style fitting
- When no gene-gene coexpression prior, epigenetic (ATAC-seq) mask, or cluster structure is available to initialize the priors and manifold constraint
- When the dataset is too small to support per-cluster EM fitting of a full 15497-gene interaction matrix

## Inputs

- scRNA-seq expression matrix with per-cell cluster labels (e.g. GSE207943, 5501 cells x 15497 genes, Phenograph labels)
- Gene-gene regulatory mask (ATAC-seq-derived and/or coexpression prior A^)
- Cluster partitions over which separate A_c matrices are fit (e.g. four Phenograph clusters)

## Outputs

- Per-cell transcriptional velocity vectors merged across clusters (one vector per cell, e.g. scKINETICS_velocity_vectors.csv with 5501 rows)
- Cluster-specific gene regulatory interaction matrices A_c (dimensions matching the 15497-gene regulatory mask, saved as .npy/.csv)
- TF ablation/GRN visualization outputs from the demo workflow

## How to apply

Load the scRNA-seq expression matrix (e.g. GSE207943: 5501 cells, 15497 genes) with cluster labels (e.g. Phenograph clusters) and a gene-gene regulatory mask from ATAC-seq/coexpression priors. For each cluster, initialize scKINETICS with the coexpression prior A^ and the manifold constraint estimated via DBSCAN-seeded truncated normal sampling of cells' future states. Run the expectation-maximization fit with noise scales sigma=5 and sigma_hat=1, capped at 20 iterations, to learn the cluster-specific interaction matrix A_c and per-cell velocities. Concatenate per-cluster velocity vectors into a merged matrix (one row per cell) and export A_c per cluster; optionally run the TF ablation experiment from the package demo to interpret regulator roles.

## Related tools

- **scKINETICS** (Core dynamical model: EM fitting of per-cell velocities and gene regulatory network using coexpression, epigenetic, and manifold priors) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Accompanying notebook demonstrating input file generation, GRN construction, EM estimation, visualizations, and TF ablation)

## Examples

```
jupyter notebook scKINETICS_demo.ipynb
```

## Evaluation signals

- Merged velocity matrix contains exactly one velocity vector per cell (e.g. 5501 rows for GSE207943)
- Each fitted A_c matrix has dimensions matching the 15497-gene regulatory mask
- EM fit terminates within the 20-iteration cap with noise scales sigma=5, sigma_hat=1
- Velocity vectors point <90 degrees (parallel) to a known biological transition axis (e.g. acinar-to-ductal metaplasia), unlike scVelo/VeloVAE/UniTVelo which give ~90-degree (orthogonal) vectors
- TF ablation experiment from the demo reproduces plausible regulator effects (e.g. factors with roles in pancreatic tumorigenesis)

## Limitations

- Priors are required: scKINETICS depends on epigenetic data, gene-gene co-expression, and phenotypic manifold constraints; without these the model cannot be properly initialized
- EM fitting is per-cluster, capped iterations (e.g. 20) may leave fits partially converged, and separate A_c matrices must be merged for whole-dataset velocities
- No changelog found for the scKINETICS repository, so version-dependent behavior differences are not documented

## Evidence

- [abstract] scKINETICS jointly learns per-cell transcriptional velocities and a governing gene regulatory network via expectation-maximization: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [intro] scKINETICS leverages priors from epigenetic data, gene-gene coexpression, and phenotypic manifold constraints: "leveraging biologically-motivated priors from epigenetic data, gene-gene co-expression, and constraints on cells’ future states imposed by the phenotypic manifold"
- [intro] EM approach learns regulator-target impacts: "an expectation-maximization approach derived to learn the impact of each regulator on its target genes"
- [readme] README demo covers the full workflow: [evidence span withheld — anchor status: external_doc]
