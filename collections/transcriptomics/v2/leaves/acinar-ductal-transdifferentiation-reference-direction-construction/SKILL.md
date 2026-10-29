---
name: acinar-ductal-transdifferentiation-reference-direction-construction
description: Use when benchmarking RNA velocity methods (scKINETICS vs scVelo/VeloVAE/UniTVelo)
  against a pseudotime-derived transdifferentiation axis in high dimensions, e.g.
  acute pancreatitis scRNA-seq.
when_to_use_negative:
- The trajectory of interest is not an acinar-to-ductal or other well-defined pseudotime-resolvable
  transdifferentiation axis — the reference direction is meaningless without a ductal
  endpoint anchor.
- Pseudotime inference cannot be anchored on the differentiation axis (e.g. no clear
  terminal state), since Palantir anchoring determines the reference directions.
- Velocity vectors and the pseudotime manifold are computed on different gene sets
  — angles would not be comparable across methods.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scKINETICS
  role: Supplies the velocity vectors whose alignment against the ADM reference directions
    is quantified via cosine angles
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook environment in which the reference-direction construction and angle
    aggregation workflow is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:0af177ca9e24fe967dea81df8115ec8c5eeb1b1b012cec8482cab9b176640ad6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# acinar-ductal-transdifferentiation-reference-direction-construction

## Summary

Constructs a per-cell reference direction along the acinar-to-ductal metaplasia (ADM) axis from pseudotime-inferred trajectory geometry, enabling angle-based evaluation of RNA velocity vectors against a biologically defined transdifferentiation manifold.

## When to use

Use when benchmarking RNA velocity methods (scKINETICS vs scVelo/VeloVAE/UniTVelo) against a pseudotime-derived transdifferentiation axis in high dimensions, e.g. acute pancreatitis scRNA-seq.

## When NOT to use

- The trajectory of interest is not an acinar-to-ductal or other well-defined pseudotime-resolvable transdifferentiation axis — the reference direction is meaningless without a ductal endpoint anchor.
- Pseudotime inference cannot be anchored on the differentiation axis (e.g. no clear terminal state), since Palantir anchoring determines the reference directions.
- Velocity vectors and the pseudotime manifold are computed on different gene sets — angles would not be comparable across methods.

## Inputs

- GSE207943 acute pancreatitis scRNA-seq count matrix (GEO)
- scanpy-preprocessed AnnData (library-size normalized, log-transformed, highly-variable-gene subset)
- scKINETICS per-cell velocity vectors (from task_002)
- splicing-based velocity vectors from scVelo, VeloVAE, and UniTVelo fit on the same gene set

## Outputs

- Per-cell ADM reference direction vectors pointing toward the ductal endpoint
- Per-cell cosine-of-angle between each method's velocity vector and the ADM direction
- Per-method mean angle and distribution summaries written to velocity_angle_concordance.csv

## How to apply

Load the GSE207943 acute pancreatitis scRNA-seq counts from GEO and preprocess with scanpy (library-size normalization, log transform, highly-variable-gene selection). Run Palantir pseudotime inference anchored on the acinar-to-ductal differentiation axis so each cell receives a pseudotime value along the transdifferentiation manifold. For each cell, build the ADM metaprogram reference direction pointing from the cell's current state toward the ductal endpoint of the inferred trajectory. This direction defines the ground-truth axis for computing per-cell cosine angles against velocity vectors (scKINETICS from the preceding task, plus the three splicing-based baselines fit on the same gene set), where scKINETICS is expected to be concordant (<90 degrees) and splicing methods near-orthogonal (~90 degrees).

## Related tools

- **scKINETICS** (Supplies the velocity vectors whose alignment against the ADM reference directions is quantified via cosine angles) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment in which the reference-direction construction and angle aggregation workflow is executed)

## Examples

```
cos_angles = {m: np.mean(np.einsum('ij,ij->i', V[m], adm_dir) / (np.linalg.norm(V[m],axis=1)*np.linalg.norm(adm_dir,axis=1))) for m in V}
```

## Evaluation signals

- scKINETICS mean cosine angle between velocity vectors and the ADM reference direction is below 90 degrees (concordant).
- scVelo, VeloVAE, and UniTVelo mean angles are approximately 90 degrees (orthogonal to the transdifferentiation direction).
- velocity_angle_concordance.csv contains per-method mean and distribution summaries as required by the schema.
- Every cell in the preprocessed GSE207943 dataset has a finite pseudotime value and a non-degenerate ADM reference direction.

## Limitations

- Angle concordance depends entirely on the Palantir anchoring choice; an incorrectly anchored ADM axis would invalidate the reference directions.
- The benchmark is specific to the acute pancreatitis ADM context; orthogonal angles in other systems may reflect biology rather than method failure.
- No changelog found for the scKINETICS repository, so pipeline versioning for reproducibility is limited.

## Evidence

- [abstract] recapitulates a well-studied axis of acinar-to-ductal transdifferentiation whilst proposing novel regulators of this process: "recapitulates a well-studied axis of acinar-to-ductal transdifferentiation whilst proposing novel regulators of this process"
- [intro] constraints on future states from the phenotypic manifold: "leveraging biologically-motivated priors from epigenetic data, gene-gene co-expression, and constraints on cells’ future states imposed by the phenotypic manifold"
- [readme] The demo shows the main functionalities of the package including input file generation, GRN construction, EM estimation, visualizations: [evidence span withheld — anchor status: external_doc]
