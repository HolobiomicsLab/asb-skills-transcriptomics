---
name: phenotypic-manifold-constraint-sampling
description: Use when fitting scKINETICS to single-cell transcriptomic data and a
  prior constraining cells' future states along the phenotypic manifold is needed
  to regularize the EM-learned gene regulatory network and velocities.
when_to_use_negative:
- When no phenotypic ordering or manifold structure is apparent in the data (e.g.,
  dissociated cells lacking a trajectory axis) — the constraint prior would impose
  spurious future-state restrictions.
- When the goal is purely an ATAC-seq-derived epigenetic prior — scKINETICS treats
  epigenetic masks and manifold constraints as separate, distinct priors.
- When the expression matrix lacks per-cell cluster labels needed to seed the manifold
  per phenograph cluster.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scKINETICS
  role: Framework that consumes the phenotypic manifold constraint as a biologically
    motivated prior when fitting the EM model of gene expression change
  repo: github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Demonstrates input file generation and GRN/EM estimation workflow including
    prior construction
  repo: github.com/dpeerlab/scKINETICS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:5edfeeeb748333751250f9756d13e62cd2f5c5d42a16257d1ee1a285d6a0ed2c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# phenotypic-manifold-constraint-sampling

## Summary

Estimates a phenotypic manifold constraint on cells' future states — one of the biologically motivated priors in scKINETICS — by seeding DBSCAN clusters on the phenotypic manifold and sampling from truncated normals to initialize the EM fit of per-cluster regulatory dynamics.

## When to use

Use when fitting scKINETICS to single-cell transcriptomic data and a prior constraining cells' future states along the phenotypic manifold is needed to regularize the EM-learned gene regulatory network and velocities.

## When NOT to use

- When no phenotypic ordering or manifold structure is apparent in the data (e.g., dissociated cells lacking a trajectory axis) — the constraint prior would impose spurious future-state restrictions.
- When the goal is purely an ATAC-seq-derived epigenetic prior — scKINETICS treats epigenetic masks and manifold constraints as separate, distinct priors.
- When the expression matrix lacks per-cell cluster labels needed to seed the manifold per phenograph cluster.

## Inputs

- scRNA-seq expression matrix (e.g., GSE207943, 5501 cells × 15497 genes)
- Phenograph cluster labels per cell
- DBSCAN seed parameters for manifold segmentation

## Outputs

- Per-cluster phenotypic manifold constraint (truncated-normal parameters) used as a scKINETICS prior
- Initialized scKINETICS runs per cluster that jointly learn A_c interaction matrices and per-cell velocity vectors

## How to apply

Load the scRNA-seq expression matrix (e.g., GSE207943, 5501 cells × 15497 genes) with Phenograph cluster labels. Within each Phenograph cluster, seed the phenotypic manifold with DBSCAN and estimate the constraint on cells' future states via truncated normal sampling seeded at the DBSCAN clusters; this prior restricts which future-state transitions are plausible during the EM fit. Supply the resulting manifold constraint alongside the gene-gene coexpression prior and (optionally) the ATAC-seq regulatory mask when initializing scKINETICS per cluster. Run the EM fit with noise scales sigma=5 and sigma_hat=1, capped at 20 iterations, so the constraint shapes the learned cluster-specific interaction matrix A_c and per-cell velocities rather than being fit itself.

## Related tools

- **scKINETICS** (Framework that consumes the phenotypic manifold constraint as a biologically motivated prior when fitting the EM model of gene expression change) — github.com/dpeerlab/scKINETICS
- **Jupyter** (Demonstrates input file generation and GRN/EM estimation workflow including prior construction) — github.com/dpeerlab/scKINETICS

## Examples

```
scKINETICS.fit(cluster_expr, coexpression_prior=A_hat, manifold_constraint=dbscan_truncated_normal(cluster_expr), sigma=5, sigma_hat=1, max_iter=20)
```

## Evaluation signals

- Each of the four Phenograph clusters yields a fitted A_c matrix whose dimensions match the 15497-gene regulatory mask.
- Merged scKINETICS_velocity_vectors.csv contains exactly one velocity vector per cell (5501 rows).
- Manifold constraint parameters are derived from DBSCAN-seeded truncated normal sampling per cluster, not fit jointly in EM.
- Resulting velocities align with the known biological axis (e.g., acinar-to-ductal metaplasia, <90 degrees) rather than orthogonal (~90 degrees) as with scVelo/VeloVAE/UniTVelo baselines.

## Limitations

- Constraint quality depends on cluster labels; mis-specified Phenograph clusters produce misleading manifold priors.
- EM fit is capped (sigma=5, sigma_hat=1, max 20 iterations), so the manifold prior can be under-converged relative to a fully fit model.
- No changelog found for the scKINETICS repository, limiting traceability of constraint-sampling implementation changes.

## Evidence

- [abstract] leveraging biologically motivated priors from epigenetic data, gene–gene coexpression, and constraints on cells' future states imposed by the phenotypic manifold: [evidence span withheld — anchor status: not_found]
- [methods] For each of the four Phenograph clusters, initialize scKINETICS with the gene-gene coexpression prior A^ and the phenotypic manifold constraint estimated via DBSCAN-seeded truncated normal sampling.: [evidence span withheld — anchor status: not_found]
- [readme] The demo shows the main functionalities of the package including input file generation, GRN construction, EM estimation, visualizations, and the TF ablation experiment.: [evidence span withheld — anchor status: external_doc]
