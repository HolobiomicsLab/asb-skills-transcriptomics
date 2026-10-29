---
name: gene-regulatory-network-inference-via-expectation-maximization
description: Use when single-cell transcriptomics data (scRNA-seq expression matrix
  with cluster labels) requires joint inference of a gene regulatory network and per-cell
  RNA velocities, especially to resolve directional dynamics like state transdifferentiation
  that standard methods miss.
when_to_use_negative:
- No epigenetic/ATAC-seq or coexpression priors are available to constrain the regulatory
  network, since the model's priors are central to its design
- The goal is velocity estimation alone without an interpretable mechanistic GRN —
  simpler tools like scVelo, VeloVAE, or UniTVelo may be preferred (though they performed
  worse on the pancreatitis benchmark)
- Data lack cluster structure or a phenotypic manifold over which future cell states
  can be constrained
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0749
tools:
- name: scKINETICS
  role: Core dynamical model fit by expectation-maximization to simultaneously learn
    per-cell transcriptional velocities and the governing gene regulatory network
  repo: github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Accompanying notebook demonstrating input file generation, GRN construction,
    EM estimation, visualizations, and TF ablation
  repo: github.com/dpeerlab/scKINETICS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:537e44e1fbcb3c2ab20f42bf7124283c82f558400cb0edd675c9feb29744259a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# gene-regulatory-network-inference-via-expectation-maximization

## Summary

scKINETICS infers a gene regulatory network and per-cell transcriptional velocities jointly by fitting a dynamical model of gene expression change via expectation-maximization, leveraging priors from epigenetic data (ATAC-seq mask), gene-gene coexpression, and phenotypic manifold constraints.

## When to use

Use when single-cell transcriptomics data (scRNA-seq expression matrix with cluster labels) requires joint inference of a gene regulatory network and per-cell RNA velocities, especially to resolve directional dynamics like state transdifferentiation that standard methods miss.

## When NOT to use

- No epigenetic/ATAC-seq or coexpression priors are available to constrain the regulatory network, since the model's priors are central to its design
- The goal is velocity estimation alone without an interpretable mechanistic GRN — simpler tools like scVelo, VeloVAE, or UniTVelo may be preferred (though they performed worse on the pancreatitis benchmark)
- Data lack cluster structure or a phenotypic manifold over which future cell states can be constrained

## Inputs

- scRNA-seq expression matrix (e.g. GSE207943: 5501 cells × 15497 genes)
- Phenograph cluster labels per cell
- Gene-gene regulatory mask (e.g. derived from ATAC-seq / epigenetic data, from an upstream task)
- Gene-gene coexpression prior A^ and phenotypic manifold constraint (DBSCAN-seeded truncated normal sampling)

## Outputs

- Cluster-specific gene regulatory network interaction matrices A_c (.npy/.csv), dimensions matching the gene count of the regulatory mask
- Merged per-cell velocity matrix (e.g. scKINETICS_velocity_vectors.csv with one velocity vector per cell, 5501 rows)

## How to apply

Load the scRNA-seq expression matrix (e.g. GSE207943: 5501 cells × 15497 genes) with Phenograph cluster labels and a gene-gene regulatory mask from ATAC-seq data (produced upstream). For each cluster, initialize scKINETICS with the gene-gene coexpression prior A^ and the phenotypic manifold constraint estimated via DBSCAN-seeded truncated normal sampling, then run the EM fit with noise scales sigma=5 and sigma_hat=1 capped at a maximum of 20 iterations to learn the cluster-specific interaction matrix A_c and per-cell velocities. Save each cluster's A_c matrix (.npy/.csv), validate its dimensions match the 15497-gene regulatory mask, and concatenate per-cluster velocity vectors into one merged matrix covering all cells.

## Related tools

- **scKINETICS** (Core dynamical model fit by expectation-maximization to simultaneously learn per-cell transcriptional velocities and the governing gene regulatory network) — github.com/dpeerlab/scKINETICS
- **Jupyter** (Accompanying notebook demonstrating input file generation, GRN construction, EM estimation, visualizations, and TF ablation) — github.com/dpeerlab/scKINETICS

## Examples

```
from scKINETICS import model; A_c, v = model.fit_em(adata, mask=atac_mask, prior_A=A_hat, manifold=dbscan_constraint, sigma=5, sigma_hat=1, max_iter=20)
```

## Evaluation signals

- Merged velocity file (scKINETICS_velocity_vectors.csv) contains exactly one velocity vector per cell (5501 rows for the pancreatitis dataset)
- Each of the four cluster-specific A_c matrices exists and its dimensions match the 15497-gene regulatory mask
- EM run respects the cap of 20 iterations with noise scales sigma=5 and sigma_hat=1 per cluster
- Velocity vectors agree with the known biology — e.g. pointing <90 degrees parallel to the acinar-to-ductal metaplasia (ADM) axis, unlike scVelo/VeloVAE/UniTVelo which were often orthogonal (~90 degrees) and largely incorrect
- Demonstrated workflows (input file generation, GRN construction, EM estimation, TF ablation) execute in the accompanying Jupyter demo notebook

## Limitations

- EM fit is capped at a maximum of 20 iterations, which may leave the interaction matrix not fully converged
- Requires biologically motivated priors (epigenetic data, gene-gene coexpression, phenotypic manifold constraints); without them the model is under-constrained
- Fit is performed per cluster, so results depend on the quality and granularity of upstream Phenograph cluster labels
- No changelog found for the scKINETICS repository, complicating version tracking of behavior changes

## Evidence

- [abstract] dynamical model fit with simultaneous learning of velocities and GRN: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [abstract] priors leveraged by the model: [evidence span withheld — anchor status: not_found]
- [intro] EM learning of regulator-target impacts: "an expectation-maximization approach derived to learn the impact of each regulator on its target genes"
- [abstract] benchmarking improvement over existing methods: "scKINETICS successfully extends and improves existing velocity approaches to generate interpretable, mechanistic models of gene regulatory dynamics"
- [readme] demo notebook contents: [evidence span withheld — anchor status: external_doc]
