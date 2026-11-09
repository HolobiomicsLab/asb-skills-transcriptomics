---
name: splicing-based-velocity-baseline-benchmarking
description: Use when you have per-cell velocity vectors from a new velocity method
  plus a pseudotime-defined state-transition axis, and need a fair head-to-head comparison
  against splicing-based methods like scVelo, VeloVAE, and UniTVelo.
when_to_use_negative:
- When no pseudotime-derived reference direction exists — cosine angles against an
  arbitrary axis are meaningless as a concordance metric.
- When the dataset lacks unspliced/spliced layer annotations required to fit splicing-based
  kinetics models like scVelo, VeloVAE, or UniTVelo.
- When evaluating regulator discovery or GRN quality rather than velocity direction
  concordance — this benchmark only scores angular alignment of per-cell velocities.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scKINETICS
  role: Source of the per-cell velocity vectors being benchmarked against splicing-based
    baselines; its output is scored for ADM-axis concordance
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook environment for running the benchmark workflow, fitting baselines,
    computing cosine angles, and writing velocity_angle_concordance.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:159d8e3ef2b45992bcba94acacb7e3355829513122c5f2df72bf10cc65d0c4cb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# splicing-based-velocity-baseline-benchmarking

## Summary

Fits splicing-kinetics RNA velocity baselines (scVelo, VeloVAE, UniTVelo) on the same preprocessed scRNA-seq gene set used for an alternative velocity method, then scores all methods by cosine-angle concordance against a pseudotime-derived reference direction (e.g. the acinar-to-ductal metaplasia axis). This isolates whether the alternative method's priors produce directionally better velocities than standard splicing-based inference.

## When to use

Use when you have per-cell velocity vectors from a new velocity method plus a pseudotime-defined state-transition axis, and need a fair head-to-head comparison against splicing-based methods like scVelo, VeloVAE, and UniTVelo.

## When NOT to use

- When no pseudotime-derived reference direction exists — cosine angles against an arbitrary axis are meaningless as a concordance metric.
- When the dataset lacks unspliced/spliced layer annotations required to fit splicing-based kinetics models like scVelo, VeloVAE, or UniTVelo.
- When evaluating regulator discovery or GRN quality rather than velocity direction concordance — this benchmark only scores angular alignment of per-cell velocities.

## Inputs

- GSE207943 acute pancreatitis scRNA-seq counts from GEO
- Preprocessed AnnData (normalized, log-transformed, HVG-selected) shared across methods
- Palantir pseudotime values anchored on the acinar-to-ductal axis
- Per-cell velocity vectors from scKINETICS (from task_002) and from scVelo, VeloVAE, UniTVelo

## Outputs

- velocity_angle_concordance.csv with per-method mean angles and distribution summaries
- Per-cell cosine-of-angle values between velocity vectors and the ADM reference direction

## How to apply

Preprocess the raw counts (GSE207943 acute pancreatitis) once with scanpy — library-size normalization, log transform, highly-variable-gene selection — so every method sees an identical matrix. Run Palantir pseudotime anchored on the acinar-to-ductal differentiation axis, then build a per-cell ADM metaprogram reference direction pointing from each cell's current state toward the ductal endpoint. Fit scVelo, VeloVAE, and UniTVelo on the same gene set used for scKINETICS (task_002 outputs), and for each cell compute the cosine of the angle between each method's velocity vector and the ADM reference. The key decision point is the 90-degree threshold: concordant methods should yield mean angles below 90 degrees, while splicing-based methods such as scVelo are expected to sit near 90 degrees (orthogonal to the transdifferentiation direction). Aggregate per-cell angles into per-method means and distributions and write velocity_angle_concordance.csv.

## Related tools

- **scKINETICS** (Source of the per-cell velocity vectors being benchmarked against splicing-based baselines; its output is scored for ADM-axis concordance) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment for running the benchmark workflow, fitting baselines, computing cosine angles, and writing velocity_angle_concordance.csv)

## Examples

```
cos_sim = np.dot(v, adm_ref)/(np.linalg.norm(v)*np.linalg.norm(adm_ref)); angle = np.degrees(np.arccos(np.clip(cos_sim, -1, 1)))
```

## Evaluation signals

- scKINETICS mean cosine angles fall below 90 degrees (concordant, pointing toward duct-like states).
- scVelo, VeloVAE, and UniTVelo mean angles are approximately 90 degrees (orthogonal to the ADM axis).
- All methods were fit on the identical gene set and the identical preprocessed matrix — verify shared HVG selection before comparing.
- velocity_angle_concordance.csv exists and contains per-method mean and distribution summary rows.

## Limitations

- Concordance is evaluated against a Palantir pseudotime-derived reference direction, so the benchmark inherits any errors in pseudotime inference or ductal-endpoint anchoring.
- The finding (scKINETICS <90°, splicing methods ~90°) was established on the GSE207943 acute pancreatitis / ADM axis; it may not transfer to other trajectories.

## Evidence

- [other] scKINETICS velocity vectors generally pointed parallel to the ADM axis towards duct-like states (<90°), whereas vectors generated from RNA splicing kinetics such as the field standard scVelo were often orthogonal (~90°): [evidence span withheld — anchor status: not_found]
- [other] Load the GSE207943 acute pancreatitis scRNA-seq counts from GEO and preprocess them (library-size normalization, log transform, highly-variable-gene selection) with scanpy: [evidence span withheld — anchor status: not_found]
- [other] Validation: verify that scKINETICS mean cosine angles are below 90 degrees (concordant with the ADM direction) while scVelo, VeloVAE, and UniTVelo yield mean angles of approximately 90 degrees: [evidence span withheld — anchor status: not_found]
- [abstract] scKINETICS successfully extends and improves existing velocity approaches to generate interpretable, mechanistic models of gene regulatory dynamics: "scKINETICS successfully extends and improves existing velocity approaches to generate interpretable, mechanistic models of gene regulatory dynamics"
- [readme] All python code and an accompanying Jupyter notebook with demonstrations are available: "All python code and an accompanying Jupyter notebook with demonstrations are available"
