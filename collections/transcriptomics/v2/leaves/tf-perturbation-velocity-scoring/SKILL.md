---
name: tf-perturbation-velocity-scoring
description: Use when you have a fitted scKINETICS model (GRN matrix A plus original
  per-cell velocities) and need single-cell TF activity scores to identify regulators
  of a state-transition process, e.g. acinar-to-ductal metaplasia.
when_to_use_negative:
- You only have raw single-cell expression data and no fitted scKINETICS model — the
  method requires the learned GRN matrix A and original velocities, not expression
  alone
- The goal is expression-based TF activity inference (e.g. regulon scoring on counts
  alone) — this score derives from velocity changes, so a low-expression TF like Ptf1a
  is exactly the case where expression-based scoring would fail
- You need true experimental knockout validation — this is purely in silico perturbation
  via zeroing GRN columns
edam_operation: http://edamontology.org/operation_2425
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: scKINETICS
  role: Provides the dynamical model used to learn the GRN and velocities, and recomputes
    per-cell velocities after each TF column of A is zeroed out (TF ablation experiment)
  repo: github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Accompanying Jupyter notebook demonstrates the package's functionalities including
    the TF ablation experiment; used to run and inspect the activity scoring workflow
  repo: github.com/dpeerlab/scKINETICS
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:b9920d7fe92eaeaca3f7a4d50f510333aff0d0fdad5cec22d5ea2b2b73b8543f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_2425
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# tf-perturbation-velocity-scoring

## Summary

An in silico transcription-factor ablation procedure built on scKINETICS: each TF's regulatory influence is removed from the learned gene regulatory network, per-cell transcriptional velocities are recomputed, and the perturbation effect is quantified as a per-cell cosine distance to assign single-cell TF activity scores.

## When to use

Use when you have a fitted scKINETICS model (GRN matrix A plus original per-cell velocities) and need single-cell TF activity scores to identify regulators of a state-transition process, e.g. acinar-to-ductal metaplasia.

## When NOT to use

- You only have raw single-cell expression data and no fitted scKINETICS model — the method requires the learned GRN matrix A and original velocities, not expression alone
- The goal is expression-based TF activity inference (e.g. regulon scoring on counts alone) — this score derives from velocity changes, so a low-expression TF like Ptf1a is exactly the case where expression-based scoring would fail
- You need true experimental knockout validation — this is purely in silico perturbation via zeroing GRN columns

## Inputs

- Gene regulatory network matrix A from a fitted scKINETICS model (from task_002)
- Original per-cell transcriptional velocities from the fitted scKINETICS model
- Cell annotations/clusters along the transition axis (e.g. acinar, transitioning, duct-like clusters)

## Outputs

- tf_activity_scores.csv — per-cell TF activity score matrix (585 TFs × 5501 cells)
- Cluster-specific TF activity patterns (e.g. Ptf1a, Sox9, Fosl2/AP1, Xbp1, Dnajc21) along the metaplasia trajectory

## How to apply

Load the gene regulatory network matrix A and the original per-cell transcriptional velocities produced by the fitted scKINETICS model. For each TF, zero out its column in A to simulate removal of that regulator's influence on its target genes, then recompute per-cell velocities under the scKINETICS dynamical model for that perturbation. Quantify the perturbation effect by computing the cosine distance between the original and perturbed velocity vectors for each cell, assembling all TFs × all cells into a TF activity score matrix (e.g. tf_activity_scores.csv with 585 TFs × 5501 cells). Inspect cluster-specific activity patterns to recover regulators: in the acute pancreatitis application, Ptf1a is elevated in acinar/early transitioning cells despite low expression, Sox9 activity increases progressively along ADM, and Fosl2/AP1 family factors, Xbp1, and Dnajc21 differentiate the two duct-like clusters. The key decision point is that the score is velocity-based, not expression-based, which is what lets it recover low-expression regulators like Ptf1a.

## Related tools

- **scKINETICS** (Provides the dynamical model used to learn the GRN and velocities, and recomputes per-cell velocities after each TF column of A is zeroed out (TF ablation experiment)) — github.com/dpeerlab/scKINETICS
- **Jupyter** (Accompanying Jupyter notebook demonstrates the package's functionalities including the TF ablation experiment; used to run and inspect the activity scoring workflow) — github.com/dpeerlab/scKINETICS

## Examples

```
A_pert = A.copy(); A_pert[:, tf_idx] = 0; v_pert = model.compute_velocity(A_pert); scores[tf] = 1 - cosine(v_orig, v_pert)
```

## Evaluation signals

- tf_activity_scores.csv contains entries for all 585 TFs across all 5501 cells
- Ptf1a activity is elevated in acinar and early transitioning cell populations with declining regulatory strength along ADM, despite low expression in the data
- Sox9 activity increases progressively along the acinar-to-ductal metaplasia trajectory
- Fosl2/AP1 family factors, Xbp1, and Dnajc21 show differential activity between the two duct-like clusters
- Recomputed perturbation velocities are well-formed per-cell vectors comparable to the original velocities (finite values, same dimensionality), enabling cosine distance computation

## Limitations

- Perturbations are in silico: zeroing a TF's column in A simulates regulator removal on target genes only within the learned network, not true experimental knockout
- Scores depend on the quality of the fitted scKINETICS model (EM-learned GRN and velocities with epigenetic, co-expression, and phenotypic manifold priors); errors in the base model propagate to activity scores
- A regulator with weak learned regulatory connections may show negligible velocity perturbation even if biologically relevant

## Evidence

- [readme] The demo shows the main functionalities of the package including input file generation, GRN construction, EM estimation, visualizations, and the TF ablation experiment.: [evidence span withheld — anchor status: external_doc]
- [abstract] an expectation-maximization approach derived to learn the impact of each regulator on its target genes: "an expectation-maximization approach derived to learn the impact of each regulator on its target genes"
- [results] recapitulates a well-studied axis of acinar-to-ductal transdifferentiation whilst proposing novel regulators of this process: "recapitulates a well-studied axis of acinar-to-ductal transdifferentiation whilst proposing novel regulators of this process"
- [intro] leveraging biologically motivated priors from epigenetic data, gene–gene coexpression, and constraints on cells' future states imposed by the phenotypic manifold: [evidence span withheld — anchor status: not_found]
