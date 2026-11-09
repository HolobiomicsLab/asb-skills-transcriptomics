---
name: cluster-wise-dynamical-model-fitting
description: Use when you have a single-cell transcriptomics dataset with cluster
  labels, a gene-gene regulatory mask (e.g. ATAC-seq-derived), and coexpression/manifold
  priors, and you need cluster-specific GRN interaction matrices plus per-cell velocity
  vectors.
when_to_use_negative:
- Input lacks cluster labels, a regulatory mask, and coexpression/manifold priors
  — the EM fit depends on these biologically motivated priors.
- The goal is a global single velocity model over all cells rather than cluster-specific
  regulatory networks (scKINETICS learns a separate A_c per cluster).
- The dataset is not single-cell transcriptomics with a definable phenotypic manifold
  over which future-state constraints can be imposed.
tools:
- name: scKINETICS
  role: Provides the dynamical model and expectation-maximization fitting routine
    that jointly learns per-cell transcriptional velocities and cluster-specific gene
    regulatory networks
  repo: github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Accompanying notebook demonstrates input file generation, GRN construction,
    EM estimation, and visualizations for the fitting workflow
  repo: github.com/dpeerlab/scKINETICS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:c349634c47f6f370e4cec989f6282cc650dbbf9e037dbaabca1b91e359689856
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cluster-wise-dynamical-model-fitting

## Summary

Fits the scKINETICS dynamical model separately per Phenograph cluster using an expectation-maximization scheme, jointly learning cluster-specific gene regulatory interaction matrices (A_c) and per-cell transcriptional velocities from scRNA-seq expression plus regulatory and manifold priors.

## When to use

Use when you have a single-cell transcriptomics dataset with cluster labels, a gene-gene regulatory mask (e.g. ATAC-seq-derived), and coexpression/manifold priors, and you need cluster-specific GRN interaction matrices plus per-cell velocity vectors.

## When NOT to use

- Input lacks cluster labels, a regulatory mask, and coexpression/manifold priors — the EM fit depends on these biologically motivated priors.
- The goal is a global single velocity model over all cells rather than cluster-specific regulatory networks (scKINETICS learns a separate A_c per cluster).
- The dataset is not single-cell transcriptomics with a definable phenotypic manifold over which future-state constraints can be imposed.

## Inputs

- GSE207943 scRNA-seq expression matrix (5501 cells x 15497 genes)
- Phenograph cluster labels for the cells
- gene-gene regulatory mask (e.g. produced from ATAC-seq data, 15497-gene dimension)
- gene-gene coexpression prior matrix A^
- phenotypic manifold constraint estimated via DBSCAN-seeded truncated normal sampling

## Outputs

- four cluster-specific A_c interaction matrices saved as .npy/.csv files (dimensions matching the 15497-gene regulatory mask)
- scKINETICS_velocity_vectors.csv — merged per-cell velocity matrix with one velocity vector per cell (5501 rows)

## How to apply

Load the GSE207943 expression matrix (5501 cells, 15497 genes) together with Phenograph cluster labels and the gene-gene regulatory mask. For each Phenograph cluster, initialize scKINETICS with the coexpression prior A^ and a phenotypic manifold constraint estimated via DBSCAN-seeded truncated normal sampling. Run the expectation-maximization fit per cluster with noise scales sigma=5 and sigma_hat=1, capped at a maximum of 20 EM iterations, to learn the cluster-specific interaction matrix A_c and per-cell velocities. Save the four fitted A_c matrices as cluster-specific .npy/.csv files, concatenate the per-cluster velocity vectors into a single merged matrix covering all 5501 cells, and save it as scKINETICS_velocity_vectors.csv. The per-cluster fitting reflects that a separate governing regulatory network is learned alongside velocities, leveraging biologically motivated priors at each cluster.

## Related tools

- **scKINETICS** (Provides the dynamical model and expectation-maximization fitting routine that jointly learns per-cell transcriptional velocities and cluster-specific gene regulatory networks) — github.com/dpeerlab/scKINETICS
- **Jupyter** (Accompanying notebook demonstrates input file generation, GRN construction, EM estimation, and visualizations for the fitting workflow) — github.com/dpeerlab/scKINETICS

## Evaluation signals

- scKINETICS_velocity_vectors.csv contains exactly one velocity vector per cell (5501 rows)
- Each of the four cluster-specific A_c matrices exists with dimensions matching the 15497-gene regulatory mask
- Velocity vectors from the fitted model point parallel to the known acinar-to-ductal metaplasia axis (angle < 90 degrees), unlike scVelo/VeloVAE/UniTVelo vectors that were often orthogonal (~90 degrees)
- EM fit completes within the capped maximum of 20 iterations with noise scales sigma=5 and sigma_hat=1

## Limitations

- Fit must be performed per cluster (four separate A_c matrices), so a single global interaction matrix is not produced
- No changelog found for the scKINETICS repository, limiting traceability of tool updates

## Evidence

- [abstract] dynamical model fit simultaneously learns velocities and GRN: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [intro] EM approach with biologically motivated priors: "leveraging biologically-motivated priors from epigenetic data, gene-gene co-expression, and constraints on cells' future states imposed by the phenotypic manifold"
- [readme] README demo covers EM estimation workflow: [evidence span withheld — anchor status: external_doc]
