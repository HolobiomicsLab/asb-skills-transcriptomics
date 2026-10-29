---
name: cosine-angle-velocity-concordance-analysis
description: 'Use when benchmarking velocity methods against a known biological axis:
  you have per-cell velocity vectors, a pseudotime-inferred endpoint, and need to
  quantify whether vectors align with or are orthogonal to the reference direction.'
when_to_use_negative:
- When no biologically grounded reference direction (e.g., pseudotime-derived endpoint)
  exists — cosine angles to an arbitrary direction are uninterpretable.
- When velocity vectors and the reference direction are not defined on the same gene
  set, since per-cell cosine computation requires identical feature spaces.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scKINETICS
  role: Provides the regulatory velocity vectors under evaluation against the ADM-axis
    reference direction
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook environment for running the angle computation and writing the concordance
    table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:0e6cfab691b47ed763addfc2a175b96c8bbabfdbfd721e5fdf8a4039048dbc46
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cosine-angle-velocity-concordance-analysis

## Summary

Evaluates RNA velocity methods by computing per-cell cosine angles between inferred velocity vectors and a pseudotime-derived reference direction (e.g., the acinar-to-ductal metaplasia axis in GSE207943 acute pancreatitis data), then aggregating per-method mean angles to test whether velocities point toward the trajectory endpoint.

## When to use

Use when benchmarking velocity methods against a known biological axis: you have per-cell velocity vectors, a pseudotime-inferred endpoint, and need to quantify whether vectors align with or are orthogonal to the reference direction.

## When NOT to use

- When no biologically grounded reference direction (e.g., pseudotime-derived endpoint) exists — cosine angles to an arbitrary direction are uninterpretable.
- When velocity vectors and the reference direction are not defined on the same gene set, since per-cell cosine computation requires identical feature spaces.

## Inputs

- Per-cell velocity vectors from each method (e.g., scKINETICS output from task_002, plus scVelo, VeloVAE, UniTVelo vectors fitted on the same gene set)
- Per-cell pseudotime values from Palantir anchored on the acinar-to-ductal differentiation axis
- Preprocessed scRNA-seq AnnData from GSE207943 (library-size normalized, log-transformed, HVG-selected)

## Outputs

- Per-method mean cosine angle to the ADM reference direction and per-cell angle distributions
- Summary table written to velocity_angle_concordance.csv

## How to apply

First derive a per-cell reference direction from pseudotime: run Palantir on scanpy-preprocessed GSE207943 acute pancreatitis counts (library-size normalization, log transform, HVG selection), anchoring the trajectory on the acinar-to-ductal differentiation axis, and construct an ADM reference direction pointing from each cell toward the ductal endpoint. Then, on the same gene set, load velocity vectors from each method being compared (scKINETICS plus splicing-based baselines scVelo, VeloVAE, UniTVelo). For each cell and each method compute the cosine of the angle between the velocity vector and the ADM reference direction, and aggregate per-cell angles into per-method means and distribution summaries. Decision criterion: a concordant method yields mean angles below 90 degrees (vectors parallel to the axis toward duct-like states), whereas methods orthogonal to the transdifferentiation direction yield mean angles of approximately 90 degrees — the expected behavior for RNA splicing kinetics methods like scVelo. Write results to a table such as velocity_angle_concordance.csv.

## Related tools

- **scKINETICS** (Provides the regulatory velocity vectors under evaluation against the ADM-axis reference direction) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment for running the angle computation and writing the concordance table)

## Examples

```
cos = np.sum(v_hat * adm_ref, axis=1) / (np.linalg.norm(v_hat, axis=1) * np.linalg.norm(adm_ref, axis=1)); angles = np.degrees(np.arccos(np.clip(cos, -1, 1)))
```

## Evaluation signals

- scKINETICS mean cosine angle is below 90 degrees (concordant, pointing toward duct-like states along the ADM axis)
- Splicing-based baselines (scVelo, VeloVAE, UniTVelo) yield mean angles of approximately 90 degrees (orthogonal to the transdifferentiation direction)
- Per-cell cosine values are bounded in [-1, 1] and correspond to angles in [0, 180] degrees
- All methods evaluated on the identical gene set so per-cell angle comparisons are valid
- Output table velocity_angle_concordance.csv contains per-method mean and distribution summaries

## Limitations

- The reference direction depends on pseudotime quality: Palantir must be anchored correctly on the acinar-to-ductal differentiation axis, or the ADM reference direction will be misoriented.
- The interpretation of ~90 degree orthogonality as a failure mode for splicing-based methods is specific to this transdifferentiation manifold; orthogonal vectors may be meaningful on other axes.
- scKINETICS relies on biologically motivated priors (epigenetic data, gene-gene co-expression, phenotypic manifold constraints); without such priors its concordance advantage may not hold.

## Evidence

- [results] scKINETICS velocity vectors generally pointed parallel to the ADM axis towards duct-like states (<90°), whereas vectors generated from RNA splicing kinetics such as the field standard scVelo were often orthogonal (~90°) to the ADM axis.: [evidence span withheld — anchor status: not_found]
- [abstract] leveraging biologically motivated priors from epigenetic data, gene–gene coexpression, and constraints on cells' future states imposed by the phenotypic manifold: [evidence span withheld — anchor status: not_found]
- [abstract] recapitulates a well-studied axis of acinar-to-ductal transdifferentiation whilst proposing novel regulators of this process: "recapitulates a well-studied axis of acinar-to-ductal transdifferentiation whilst proposing novel regulators of this process"
- [readme] The demo shows the main functionalities of the package including input file generation, GRN construction, EM estimation, visualizations, and the TF ablation experiment.: [evidence span withheld — anchor status: external_doc]
