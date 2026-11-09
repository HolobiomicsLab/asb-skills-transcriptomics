---
name: cosine-distance-activity-quantification
description: Use when you have a fitted scKINETICS model (GRN matrix A and per-cell
  transcriptional velocities) and need per-cell, per-TF activity scores via in silico
  knockout rather than expression-based TF activity inference.
when_to_use_negative:
- When no fitted scKINETICS model exists — velocities and the GRN matrix A must come
  from the EM fit; this skill cannot run on raw counts alone
- When the goal is TF activity from expression or motif enrichment (e.g., SCENIC-style
  regulon scoring) rather than velocity-perturbation-based activity
- When the regulatory network is unknown or not parameterized per regulator column,
  so individual TF influence cannot be zeroed out
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scKINETICS
  role: Fits the dynamical model that jointly learns per-cell velocities and the GRN
    matrix A whose columns are ablated, and recomputes velocities under each TF knockout
  repo: github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook environment used for running the TF ablation/velocity-perturbation
    workflow and assembling activity score outputs
  repo: http://github.com/dpeerlab/scKINETICS
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:abd645f8e6776e27e320df8f958a65dc619de336d22af231ac741506cc1e7c50
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cosine-distance-activity-quantification

## Summary

Quantifies per-cell transcription factor activity under the scKINETICS framework by recomputing velocities after in silico zeroing of each TF's column in the learned GRN matrix A, then scoring the perturbation effect as the cosine distance between original and perturbed velocity vectors across all cells.

## When to use

Use when you have a fitted scKINETICS model (GRN matrix A and per-cell transcriptional velocities) and need per-cell, per-TF activity scores via in silico knockout rather than expression-based TF activity inference.

## When NOT to use

- When no fitted scKINETICS model exists — velocities and the GRN matrix A must come from the EM fit; this skill cannot run on raw counts alone
- When the goal is TF activity from expression or motif enrichment (e.g., SCENIC-style regulon scoring) rather than velocity-perturbation-based activity
- When the regulatory network is unknown or not parameterized per regulator column, so individual TF influence cannot be zeroed out

## Inputs

- Gene regulatory network matrix A from a fitted scKINETICS model (regulators x target genes)
- Original per-cell transcriptional velocities from the fitted scKINETICS dynamical model
- List of transcription factors to ablate (e.g., all 585 TFs)
- Per-cell annotation of cluster/state (e.g., 5501 cells spanning acinar, transitioning, and duct-like populations)

## Outputs

- Per-cell TF activity score matrix (TFs x cells), e.g., tf_activity_scores.csv with 585 TFs x 5501 cells
- Cluster-specific activity patterns for individual TFs (e.g., Ptf1a, Sox9, Fosl2, Xbp1, Dnajc21)

## How to apply

Load the gene regulatory network matrix A and the original per-cell transcriptional velocities from the fitted scKINETICS EM model. For each of the 585 transcription factors, zero out that TF's column in A to simulate removal of its regulatory influence on target genes, then recompute per-cell velocities using the scKINETICS dynamical model under each perturbation. Quantify the perturbation effect per cell as the cosine distance between the original and perturbed velocity vectors, and assemble the resulting scores into a TF x cell activity matrix (585 TFs x 5501 cells in the acute pancreatitis/ADM application). Interpretation rests on activity being decoupled from expression: Ptf1a scores are elevated in acinar/early transitioning cells despite low expression, Sox9 activity rises progressively along acinar-to-ductal metaplasia, and Fosl2/AP1, Xbp1, and Dnajc21 differentiate the two duct-like clusters.

## Related tools

- **scKINETICS** (Fits the dynamical model that jointly learns per-cell velocities and the GRN matrix A whose columns are ablated, and recomputes velocities under each TF knockout) — github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment used for running the TF ablation/velocity-perturbation workflow and assembling activity score outputs) — http://github.com/dpeerlab/scKINETICS

## Examples

```
A_perturbed = A.copy(); A_perturbed[:, tf_idx] = 0; v_pert = model.recompute_velocity(A_perturbed); d[tf_idx, :] = cosine(v_orig, v_pert, axis=0)
```

## Evaluation signals

- tf_activity_scores.csv must contain entries for all 585 TFs across all 5501 cells (no missing TF/cell combinations)
- Ptf1a activity is elevated in acinar and early transitioning cell populations despite low Ptf1a expression in the data
- Sox9 activity increases progressively along the acinar-to-ductal metaplasia trajectory
- Fosl2/AP1 family factors, Xbp1, and Dnajc21 show distinct (differential) activity between the two duct-like clusters
- Recomputed activity patterns reproduce the reported transdifferentiation regulatory landscape from the paper

## Limitations

- Activity scores reflect the effect of removing a regulator's influence in the learned GRN, so they are bounded by the quality and priors (epigenetic data, co-expression, phenotypic manifold constraints) that shaped matrix A
- Perturbation requires recomputing velocities for every TF (585 fits), which scales linearly with the number of regulators screened
- Lowly expressed TFs can still show high activity (e.g., Ptf1a), so activity scores should not be sanity-checked against expression levels

## Evidence

- [other] task card, workflow: [evidence span withheld — anchor status: not_found]
- [readme] README, demo: [evidence span withheld — anchor status: external_doc]
- [abstract] article abstract, EM model: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [other] task card, finding: [evidence span withheld — anchor status: not_found]
