---
name: pseudotime-trajectory-inference
description: Use when scRNA-seq captures a continuous state-transition process (e.g.,
  acinar-to-ductal transdifferentiation) and you need a pseudotime-anchored reference
  direction to benchmark velocity vectors against, as in the ADM-axis concordance
  task.
when_to_use_negative:
- Data represent discrete, disconnected cell populations with no continuous transdifferentiation
  axis — diffusion-map pseudotime anchoring will be meaningless
- You need regulatory network or velocity inference itself; this skill supplies the
  manifold reference, while tools like scKINETICS fit the per-cell velocities and
  GRN
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Palantir
  role: Diffusion-map pseudotime inference anchored on the acinar-to-ductal axis
- name: scanpy
  role: Library-size normalization, log transform, and highly-variable-gene selection
    before pseudotime
- name: Jupyter
  role: Notebook environment for running the pseudotime and angle-concordance workflow
- name: scKINETICS
  role: 'Downstream consumer: its velocity vectors are scored against the pseudotime-derived
    ADM reference direction'
  repo: github.com/dpeerlab/scKINETICS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:ae74bd11455dd301afba0952882d8510fa1991b253030a95c144eb1c12ba310c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - Palantir
  - scKINETICS
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# pseudotime-trajectory-inference

## Summary

Infer per-cell pseudotime along a transdifferentiation manifold (e.g., acinar-to-ductal metaplasia in GSE207943 acute pancreatitis scRNA-seq) with Palantir, then convert the trajectory into per-cell reference directions used to evaluate or constrain RNA velocity methods such as scKINETICS.

## When to use

Use when scRNA-seq captures a continuous state-transition process (e.g., acinar-to-ductal transdifferentiation) and you need a pseudotime-anchored reference direction to benchmark velocity vectors against, as in the ADM-axis concordance task.

## When NOT to use

- Data represent discrete, disconnected cell populations with no continuous transdifferentiation axis — diffusion-map pseudotime anchoring will be meaningless
- You need regulatory network or velocity inference itself; this skill supplies the manifold reference, while tools like scKINETICS fit the per-cell velocities and GRN

## Inputs

- Raw scRNA-seq count matrix from GEO (GSE207943 acute pancreatitis) as scanpy-loadable AnnData
- Cell-type/state annotations identifying the start (acinar) and endpoint (duct-like) populations of the trajectory
- Set of highly variable genes / same gene set shared with velocity methods

## Outputs

- Per-cell pseudotime values along the transdifferentiation manifold
- Per-cell ADM reference direction vectors pointing toward the ductal endpoint
- Per-cell cosines of the angle between each velocity method's vectors and the ADM axis, with per-method mean/distribution summaries (e.g., velocity_angle_concordance.csv)

## How to apply

Preprocess the raw counts (e.g., GEO accession GSE207943 acute pancreatitis) with scanpy: library-size normalization, log transform, and highly-variable-gene selection. Run Palantir pseudotime inference, anchoring the trajectory on the biological axis of interest (acinar-to-ductal differentiation), so every cell receives a pseudotime value along the transdifferentiation manifold. Construct a per-cell reference direction (e.g., the ADM metaprogram vector) pointing from each cell's current state toward the ductal endpoint of the inferred trajectory. This manifold constraint is the same prior scKINETICS exploits — 'constraints on cells' future states imposed by the phenotypic manifold' — and the resulting directions let you compute per-cell cosines between a method's velocity vectors (scKINETICS, scVelo, VeloVAE, UniTVelo) and the trajectory axis; concordant methods yield mean angles below 90 degrees while splicing-based methods drift toward ~90 degrees (orthogonal). Aggregate per-cell angles into per-method means and distributions for downstream comparison (e.g., velocity_angle_concordance.csv).

## Related tools

- **Palantir** (Diffusion-map pseudotime inference anchored on the acinar-to-ductal axis)
- **scanpy** (Library-size normalization, log transform, and highly-variable-gene selection before pseudotime)
- **Jupyter** (Notebook environment for running the pseudotime and angle-concordance workflow)
- **scKINETICS** (Downstream consumer: its velocity vectors are scored against the pseudotime-derived ADM reference direction) — github.com/dpeerlab/scKINETICS

## Examples

```
import scanpy as sc; sc.pp.normalize_total(adata); sc.pp.log1p(adata); sc.pp.highly_variable_genes(adata, n_top_genes=2000)  # preprocess GSE207943 counts before Palantir pseudotime
```

## Evaluation signals

- Every cell receives a finite, monotonic-consistent pseudotime value along the ADM axis
- Mean angle between scKINETICS velocity vectors and the ADM reference direction is <90 degrees (concordant)
- Mean angles for scVelo, VeloVAE, and UniTVelo come out ~90 degrees (orthogonal to the transdifferentiation direction)
- All methods are fit on the same gene set as the pseudotime preprocessing so cosine comparisons are apples-to-apples
- Angle table written to velocity_angle_concordance.csv includes per-method means and distribution summaries

## Limitations

- Reference direction quality depends on correctly anchoring Palantir on the acinar-to-ductal differentiation axis; a mis-specified root or endpoint will rotate the ADM vectors and invalidate the angle concordance
- Comparison is only valid when all velocity methods (scKINETICS and the splicing-based baselines) are run on the same preprocessed gene set

## Evidence

- [other] Run Palantir pseudotime inference, anchoring the trajectory on the acinar-to-ductal differentiation axis so each cell receives a pseudotime value along the transdifferentiation manifold.: [evidence span withheld — anchor status: not_found]
- [intro] leveraging biologically-motivated priors from epigenetic data, gene-gene co-expression, and constraints on cells’ future states imposed by the phenotypic manifold: "leveraging biologically-motivated priors from epigenetic data, gene-gene co-expression, and constraints on cells’ future states imposed by the phenotypic manifold"
- [abstract] recapitulates a well-studied axis of acinar-to-ductal transdifferentiation whilst proposing novel regulators of this process: "recapitulates a well-studied axis of acinar-to-ductal transdifferentiation whilst proposing novel regulators of this process, including factors with previously appreciated roles in driving pancreatic"
- [results] scKINETICS velocity vectors generally pointed parallel to the ADM axis towards duct-like states (<90°), whereas vectors generated from RNA splicing kinetics such as the field standard scVelo were often orthogonal (~90°): [evidence span withheld — anchor status: not_found]
