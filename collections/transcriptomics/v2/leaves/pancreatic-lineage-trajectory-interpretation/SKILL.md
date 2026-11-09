---
name: pancreatic-lineage-trajectory-interpretation
description: Use when a fitted scKINETICS model (gene regulatory network matrix A
  plus per-cell transcriptional velocities) is available and TF-level regulator activity
  across a pancreatic lineage transition such as acinar-to-ductal metaplasia must
  be inferred.
when_to_use_negative:
- Input is raw single-cell expression data without a fitted scKINETICS model — the
  knockout screen requires the GRN matrix A and per-cell velocities from a prior EM
  fit
- The analysis goal is TF activity from co-expression or motif enrichment alone (no
  velocity/dynamical model), where velocity-perturbation scoring is not applicable
- The dataset contains no pancreatic lineage transition (e.g., ADM) or phenotypic
  manifold along which to interpret cluster-specific activity patterns
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scKINETICS
  role: Dynamical model whose fitted GRN matrix A and per-cell velocities are perturbed
    (TF column zeroed) to recompute velocities and score TF activity by cosine distance;
    the package demo includes the TF ablation experiment
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook environment for running the scKINETICS demo (input file generation,
    GRN construction, EM estimation, visualizations, TF ablation) and assembling tf_activity_scores.csv
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:dce21e95d4c7664d47e52cbb5ce5df30c4421a96171a34872d77b1a3423a4f93
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# pancreatic-lineage-trajectory-interpretation

## Summary

Quantify transcription-factor regulatory activity along a pancreatic acinar-to-ductal metaplasia (ADM) trajectory by simulating in silico TF knockouts in a fitted scKINETICS dynamical model and scoring per-cell velocity perturbations, recovering regulators (e.g., Ptf1a) whose influence is not apparent from expression alone.

## When to use

Use when a fitted scKINETICS model (gene regulatory network matrix A plus per-cell transcriptional velocities) is available and TF-level regulator activity across a pancreatic lineage transition such as acinar-to-ductal metaplasia must be inferred.

## When NOT to use

- Input is raw single-cell expression data without a fitted scKINETICS model — the knockout screen requires the GRN matrix A and per-cell velocities from a prior EM fit
- The analysis goal is TF activity from co-expression or motif enrichment alone (no velocity/dynamical model), where velocity-perturbation scoring is not applicable
- The dataset contains no pancreatic lineage transition (e.g., ADM) or phenotypic manifold along which to interpret cluster-specific activity patterns

## Inputs

- Gene regulatory network matrix A from a fitted scKINETICS model (585 TFs as regulator columns)
- Original per-cell transcriptional velocity vectors for 5501 cells from the fitted scKINETICS model
- Cell cluster/phenotypic-manifold labels along the acinar-to-ductal metaplasia axis for cluster-specific activity interpretation

## Outputs

- tf_activity_scores.csv: per-cell TF activity matrix (585 TFs x 5501 cells) from cosine-distance scoring
- Cluster-specific activity patterns for Ptf1a, Sox9, Fosl2/AP1 factors, Xbp1, and Dnajc21
- Ranking of known and novel regulators of acinar-to-ductal metaplasia

## How to apply

Load the gene regulatory network matrix A and the original per-cell transcriptional velocities produced by the fitted scKINETICS model. For each of the 585 transcription factors, zero out its column in A to simulate removal of that regulator's influence on its target genes, then recompute per-cell velocities under each perturbation using the scKINETICS dynamical model. Quantify each perturbation's effect by computing the cosine distance between original and perturbed velocity vectors for each of the 5501 cells, assembling the results into a 585 TF x 5501 cell activity-score matrix (tf_activity_scores.csv). Interpret cluster-specific patterns: Ptf1a activity should be elevated in acinar and early transitioning populations, Sox9 activity should progressively increase along ADM, and Fosl2/AP1 family factors, Xbp1, and Dnajc21 should show differential activity between the two duct-like clusters. The rationale is that regulatory activity is measured as sensitivity of the learned dynamics to a regulator's removal, so low-expression TFs like Ptf1a can still be recovered when their regulatory impact is strong.

## Related tools

- **scKINETICS** (Dynamical model whose fitted GRN matrix A and per-cell velocities are perturbed (TF column zeroed) to recompute velocities and score TF activity by cosine distance; the package demo includes the TF ablation experiment) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment for running the scKINETICS demo (input file generation, GRN construction, EM estimation, visualizations, TF ablation) and assembling tf_activity_scores.csv)

## Examples

```
A_pert = A.copy(); A_pert[:, tf_idx] = 0.0; v_pert = model.recompute_velocities(A_pert); score = 1 - cosine_similarity(v, v_pert)
```

## Evaluation signals

- tf_activity_scores.csv contains entries for all 585 TFs across all 5501 cells (585 x 5501 matrix)
- Ptf1a activity is elevated in acinar and early transitioning cell populations, despite low Ptf1a expression in the data
- Sox9 activity increases progressively along the acinar-to-ductal metaplasia trajectory
- Fosl2/AP1 family factors, Xbp1, and Dnajc21 show differential activity between the two duct-like clusters
- Cosine distances between original and perturbed velocity vectors are finite, non-negative, and fall in a consistent range across perturbations

## Limitations

- Regulator recovery depends on priors (epigenetic data, gene-gene co-expression, phenotypic manifold constraints) built into the scKINETICS fit; a TF absent from these priors or the GRN cannot be ablated
- Activity scores reflect sensitivity of learned dynamics to a regulator's removal, not expression level — a TF like Ptf1a can be a top regulator while having low expression, so expression-based interpretation is misleading
- No changelog found for the scKINETICS repository, so version drift in the ablation workflow cannot be tracked from the repo alone

## Evidence

- [abstract] recapitulates a well-studied axis of acinar-to-ductal transdifferentiation whilst proposing novel regulators of this process: [evidence span withheld — anchor status: not_found]
- [intro] a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network: "We introduce scKINETICS (Key regulatory Interaction NETwork for Inferring Cell Speed), a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell"
- [readme] The demo shows the main functionalities of the package including input file generation, GRN construction, EM estimation, visualizations, and the TF ablation experiment.: [evidence span withheld — anchor status: external_doc]
- [abstract] leveraging biologically-motivated priors from epigenetic data, gene-gene co-expression, and constraints on cells' future states imposed by the phenotypic manifold: [evidence span withheld — anchor status: not_found]
