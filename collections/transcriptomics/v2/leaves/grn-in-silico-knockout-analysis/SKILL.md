---
name: grn-in-silico-knockout-analysis
description: Use when a fitted scKINETICS model (GRN matrix A plus per-cell velocities)
  is available and you need per-cell TF activity scores across all regulators to identify
  regulators driving a transition, such as acinar-to-ductal metaplasia.
when_to_use_negative:
- No fitted scKINETICS model exists yet — the GRN matrix A and original velocities
  must first be produced by fitting the scKINETICS dynamical model (EM estimation);
  use the model-fitting skill first.
- 'The goal is differential gene expression analysis rather than regulator activity:
  cosine distance of velocity perturbation measures regulatory influence, not expression
  level, so it will not summarize expression (e.g. Ptf1a is recovered by activity
  despite low expression).'
- The perturbation of interest is a multi-gene or pathway-level perturbation rather
  than removal of a single TF's column in the GRN.
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3308
tools:
- name: scKINETICS
  role: Provides the dynamical model used to recompute per-cell velocities after zeroing
    out each TF's column in the learned GRN matrix A, and supplies the GRN and original
    velocities that constitute the inputs to the knockout screen.
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Environment for running the TF ablation experiment and visualizations; the
    repository's demo includes the TF ablation experiment workflow.
  repo: http://github.com/dpeerlab/scKINETICS
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:a5406986f44ebea12455b033ebc6f3f780372fdd1e2fbb8ed0152ee5d281efb1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# grn-in-silico-knockout-analysis

## Summary

In silico transcription factor knockout using scKINETICS: zero out each TF's column in the learned gene regulatory network, recompute per-cell velocities, and score each regulator's activity by the cosine distance between original and perturbed velocity vectors to recover regulators of acinar-to-ductal metaplasia.

## When to use

Use when a fitted scKINETICS model (GRN matrix A plus per-cell velocities) is available and you need per-cell TF activity scores across all regulators to identify regulators driving a transition, such as acinar-to-ductal metaplasia.

## When NOT to use

- No fitted scKINETICS model exists yet — the GRN matrix A and original velocities must first be produced by fitting the scKINETICS dynamical model (EM estimation); use the model-fitting skill first.
- The goal is differential gene expression analysis rather than regulator activity: cosine distance of velocity perturbation measures regulatory influence, not expression level, so it will not summarize expression (e.g. Ptf1a is recovered by activity despite low expression).
- The perturbation of interest is a multi-gene or pathway-level perturbation rather than removal of a single TF's column in the GRN.

## Inputs

- Gene regulatory network matrix A learned by the fitted scKINETICS model (regulator × target gene weights)
- Original per-cell transcriptional velocities produced by the fitted scKINETICS model
- Cell annotations / cluster labels for the 5501 cells along the acinar-to-ductal metaplasia trajectory
- List of 585 transcription factors to ablate

## Outputs

- tf_activity_scores.csv: per-cell TF activity score matrix (585 TFs × 5501 cells)
- Cluster-specific activity patterns for TFs of interest (e.g. Ptf1a, Sox9, Fosl2/AP1, Xbp1, Dnajc21)
- Differential TF activity comparisons between cell states (e.g. the two duct-like clusters)

## How to apply

Load the gene regulatory network matrix A and the original per-cell transcriptional velocities from the fitted scKINETICS model (e.g. output of the prior EM fitting step in task_002). For each of the 585 transcription factors, zero out its column in A to simulate removal of that regulator's influence on its target genes, then recompute the per-cell velocities under each perturbation using the scKINETICS dynamical model. Quantify the perturbation effect by computing the cosine distance between original and perturbed velocity vectors for each of the 5501 cells, assembling the resulting 585 TFs × 5501 cells activity matrix into tf_activity_scores.csv. The rationale is that a regulator whose removal strongly changes the velocity field in a given cell is inferred to be active there — this recovers factors like Ptf1a that are active despite low expression. Finally, map scores onto cell clusters/states to examine cluster-specific patterns (e.g. Ptf1a elevated in acinar/early cells, Sox9 increasing along ADM, and Fosl2/AP1, Xbp1, Dnajc21 differing between duct-like clusters).

## Related tools

- **scKINETICS** (Provides the dynamical model used to recompute per-cell velocities after zeroing out each TF's column in the learned GRN matrix A, and supplies the GRN and original velocities that constitute the inputs to the knockout screen.) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Environment for running the TF ablation experiment and visualizations; the repository's demo includes the TF ablation experiment workflow.) — http://github.com/dpeerlab/scKINETICS

## Examples

```
A[tf_idx, :] = 0; v_pert = sklearn_fit(A, inputs)  # recompute velocities; scores[tf] = cosine(original_v, v_pert)
```

## Evaluation signals

- tf_activity_scores.csv contains entries for all 585 TFs across all 5501 cells.
- Ptf1a activity is elevated in acinar and early transitioning cell populations, recovering the canonical acinar TF despite its low expression in the data.
- Sox9 activity increases progressively along the acinar-to-ductal metaplasia trajectory.
- Fosl2/AP1 family factors, Xbp1, and Dnajc21 show distinct activity between the two duct-like clusters, reproducing the reported transdifferentiation regulatory landscape.
- For each TF ablation, the recomputed velocities are finite and the cosine distance to the original velocities falls within the expected [0, 2] range.

## Limitations

- TF activity is inferred from velocity perturbation, not expression, so factors with low expression (like Ptf1a) are only detectable through their learned regulatory strength; the approach depends on the quality of the EM-learned GRN.
- The knockout is simulated in silico by zeroing a single TF's column in A; it does not capture compensatory network rewiring that might occur in a real genetic perturbation.
- No changelog was found for the scKINETICS repository, so version-specific behavior of the ablation workflow may be undocumented.

## Evidence

- [other] For each of the 585 transcription factors, zero out its column in A to simulate removal of that regulator's influence on its target genes.: [evidence span withheld — anchor status: not_found]
- [other] Quantify the effect of each TF perturbation by computing the cosine distance between the original and perturbed velocity vectors for each of the 5501 cells.: [evidence span withheld — anchor status: not_found]
- [readme] The demo shows the main functionalities of the package including input file generation, GRN construction, EM estimation, visualizations, and the TF ablation experiment.: [evidence span withheld — anchor status: external_doc]
- [intro] scKINETICS learns the impact of each regulator on its target genes through an expectation-maximization approach.: "an expectation-maximization approach derived to learn the impact of each regulator on its target genes"
- [abstract] The paper reports recovering regulators of acinar-to-ductal transdifferentiation, including factors with roles in pancreatic tumorigenesis.: "recapitulates a well-studied axis of acinar-to-ductal transdifferentiation whilst proposing novel regulators of this process"
