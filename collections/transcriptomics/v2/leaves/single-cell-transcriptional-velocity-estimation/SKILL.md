---
name: single-cell-transcriptional-velocity-estimation
description: 'Use when you have scRNA-seq expression data (e.g. GSE207943: 5501 cells
  x 15497 genes) plus a gene-gene regulatory mask or coexpression prior and Phenograph
  cluster labels, and need per-cell velocity vectors and cluster-specific regulatory
  interaction matrices.'
when_to_use_negative:
- Your only input is raw spliced/unspliced count matrices and you have no regulatory
  mask, coexpression prior, or epigenetic data — scVelo-family methods are the standard
  choice there (though they yielded ~90-degree, largely incorrect vectors on the pancreatitis
  ADM axis)
- You need a ready-made tool with a changelog/maintenance history — no changelog was
  found for the scKINETICS repository
- The goal is not velocity/GRN inference (e.g. clustering or trajectory pseudotime
  without dynamical modeling)
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scKINETICS
  role: Runs the expectation-maximization fit that simultaneously learns per-cell
    transcriptional velocities and the governing gene regulatory network, using epigenetic,
    coexpression, and phenotypic manifold priors
  repo: github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Accompanying notebook demonstrating input file generation, GRN construction,
    EM estimation, visualizations, and the TF ablation experiment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:ac010b9e7d3d721e38db025136ca7ae2658f0dcc21c29f666e6dc6bf1563a6bf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# single-cell-transcriptional-velocity-estimation

## Summary

Estimate per-cell transcriptional velocities with scKINETICS, an expectation-maximization dynamical model that simultaneously learns per-cell velocities and a governing gene regulatory network from biologically-motivated priors (epigenetic data, gene-gene coexpression, phenotypic manifold constraints). Applied to the GSE207943 pancreatitis dataset it recapitulates the acinar-to-ductal transdifferentiation axis where scVelo, VeloVAE, and UniTVelo produced largely orthogonal velocity vectors.

## When to use

Use when you have scRNA-seq expression data (e.g. GSE207943: 5501 cells x 15497 genes) plus a gene-gene regulatory mask or coexpression prior and Phenograph cluster labels, and need per-cell velocity vectors and cluster-specific regulatory interaction matrices.

## When NOT to use

- Your only input is raw spliced/unspliced count matrices and you have no regulatory mask, coexpression prior, or epigenetic data — scVelo-family methods are the standard choice there (though they yielded ~90-degree, largely incorrect vectors on the pancreatitis ADM axis)
- You need a ready-made tool with a changelog/maintenance history — no changelog was found for the scKINETICS repository
- The goal is not velocity/GRN inference (e.g. clustering or trajectory pseudotime without dynamical modeling)

## Inputs

- scRNA-seq expression matrix (e.g. GSE207943: 5501 cells x 15497 genes)
- Phenograph cluster labels
- Gene-gene regulatory mask derived from ATAC-seq / epigenetic data
- Gene-gene coexpression prior matrix A^
- Phenotypic manifold constraint (DBSCAN-seeded truncated normal sampling)

## Outputs

- Cluster-specific gene regulatory interaction matrices A_c (.npy/.csv, dimensions matching the 15497-gene mask)
- Merged per-cell velocity matrix (one velocity vector per cell, 5501 rows, e.g. scKINETICS_velocity_vectors.csv)

## How to apply

Load the scRNA-seq expression matrix together with Phenograph cluster labels and a gene-gene regulatory mask (e.g. derived from ATAC-seq). For each cluster, initialize scKINETICS with the gene-gene coexpression prior A^ and a phenotypic manifold constraint estimated via DBSCAN-seeded truncated normal sampling. Run the expectation-maximization fit per cluster with noise scales sigma=5 and sigma_hat=1, capped at a maximum of 20 EM iterations, to learn the cluster-specific interaction matrix A_c and per-cell velocities. Save each A_c as .npy/.csv sized to the 15497-gene mask, and concatenate per-cell velocity vectors into a single merged matrix covering all 5501 cells. The rationale is that scKINETICS's EM approach learns the impact of each regulator on its target genes while priors constrain cells' future states along the phenotypic manifold.

## Related tools

- **scKINETICS** (Runs the expectation-maximization fit that simultaneously learns per-cell transcriptional velocities and the governing gene regulatory network, using epigenetic, coexpression, and phenotypic manifold priors) — github.com/dpeerlab/scKINETICS
- **Jupyter** (Accompanying notebook demonstrating input file generation, GRN construction, EM estimation, visualizations, and the TF ablation experiment)

## Evaluation signals

- scKINETICS_velocity_vectors.csv contains exactly one velocity vector per cell (5501 rows)
- Each of the four Phenograph-cluster A_c matrices exists and its dimensions match the 15497-gene regulatory mask
- EM fits stop at or below the maximum of 20 iterations with sigma=5 and sigma_hat=1 noise scales
- Fitted velocity vectors point generally parallel to the known acinar-to-ductal metaplasia axis (angle < 90 degrees toward duct-like states), unlike scVelo/VeloVAE/UniTVelo which were often orthogonal (~90 degrees)
- Demo notebook workflows (input file generation, GRN construction, EM estimation, visualization, TF ablation) execute end to end

## Limitations

- Performance depends on quality of the priors: the epigenetic (ATAC-seq) mask, gene-gene coexpression prior, and phenotypic manifold constraint must be available and accurate
- A per-cluster EM fit is required (four Phenograph clusters in the pancreatitis case), so results depend on cluster labels and initialization (DBSCAN-seeded truncated normal sampling)
- No changelog found for the scKINETICS repository, indicating limited versioning/maintenance signals

## Evidence

- [readme] We introduce scKINETICS (Key regulatory Interaction NETwork for Inferring Cell Speed), a dynamical model of gene expression change: "We introduce scKINETICS (Key regulatory Interaction NETwork for Inferring Cell Speed), a dynamical model of gene expression change"
- [intro] an expectation-maximization approach derived to learn the impact of each regulator on its target genes: "an expectation-maximization approach derived to learn the impact of each regulator on its target genes"
- [abstract] leveraging biologically motivated priors from epigenetic data, gene–gene coexpression, and constraints on cells' future states imposed by the phenotypic manifold: [evidence span withheld — anchor status: not_found]
- [other] Run the scKINETICS expectation-maximization fit on each cluster with noise scales sigma=5 and sigma_hat=1, capped at a maximum of 20 iterations: [evidence span withheld — anchor status: not_found]
- [abstract] recapitulates a well-studied axis of acinar-to-ductal transdifferentiation whilst proposing novel regulators of this process: "recapitulates a well-studied axis of acinar-to-ductal transdifferentiation whilst proposing novel regulators of this process"
