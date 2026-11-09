---
name: per-cell-regulator-activity-comparison-across-clusters
description: Use when you have a fitted scKINETICS model (GRN matrix A + per-cell
  velocities) and need TF-level, per-cell activity scores to identify regulators whose
  influence varies across clusters, e.g. acinar vs transitioning vs duct-like populations.
when_to_use_negative:
- No fitted scKINETICS model is available — the GRN matrix A and original velocities
  must first be produced by model fitting (EM estimation), so this skill cannot run
  on raw expression data alone
- The question is differential gene expression between clusters rather than regulator
  influence — activity scores measure regulatory impact via velocity perturbation,
  not expression abundance (e.g. Ptf1a scores are high despite low expression)
- You need perturbation effects on cell fate/phenotype trajectories rather than per-cell
  regulator activity scores from velocity ablation
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3315
tools:
- name: scKINETICS
  role: Provides the fitted GRN matrix and dynamical model used to zero TF columns,
    recompute perturbed per-cell velocities, and run the TF ablation experiment
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook environment for the demonstration workflow including the TF ablation
    experiment and visualization of activity patterns
  repo: http://github.com/dpeerlab/scKINETICS
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:8ddbf9f24236462695326d3bdb6a35f4abdcad42ef945341f91629db216c4f6c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# per-cell-regulator-activity-comparison-across-clusters

## Summary

Quantifies each transcription factor's regulatory influence at single-cell resolution by in silico knockout: zeroing the TF's column in the learned GRN, recomputing scKINETICS velocities, and scoring perturbation effects as per-cell cosine distance between original and perturbed velocity vectors, then comparing activity patterns across cell clusters.

## When to use

Use when you have a fitted scKINETICS model (GRN matrix A + per-cell velocities) and need TF-level, per-cell activity scores to identify regulators whose influence varies across clusters, e.g. acinar vs transitioning vs duct-like populations.

## When NOT to use

- No fitted scKINETICS model is available — the GRN matrix A and original velocities must first be produced by model fitting (EM estimation), so this skill cannot run on raw expression data alone
- The question is differential gene expression between clusters rather than regulator influence — activity scores measure regulatory impact via velocity perturbation, not expression abundance (e.g. Ptf1a scores are high despite low expression)
- You need perturbation effects on cell fate/phenotype trajectories rather than per-cell regulator activity scores from velocity ablation

## Inputs

- Gene regulatory network matrix A (regulator x target weights) from the fitted scKINETICS model
- Original per-cell transcriptional velocities produced by the fitted scKINETICS model (from the prior fitting task)
- List of transcription factors to ablate (e.g. 585 TFs)
- Cell annotation / cluster labels for per-cluster activity comparisons

## Outputs

- Per-cell TF activity score matrix (TFs x cells), e.g. tf_activity_scores.csv with entries for all 585 TFs across all 5501 cells
- Cluster-specific activity patterns for TFs of interest (e.g. Ptf1a, Sox9, Fosl2/AP1 family, Xbp1, Dnajc21)

## How to apply

Load the gene regulatory network matrix A and original per-cell transcriptional velocities from the fitted scKINETICS model. For each TF (e.g. all 585 TFs across 5501 cells), zero out its column in A to simulate removal of that regulator's influence on its target genes, then recompute per-cell velocities under the perturbation using the scKINETICS dynamical model. Quantify each perturbation's effect by computing the cosine distance between original and perturbed velocity vectors for every cell, and assemble the resulting TF x cell activity score matrix (e.g. tf_activity_scores.csv). The rationale is that a TF with strong influence on its targets in a given cell produces a large velocity shift when ablated, so the cosine distance serves as a cell-resolved regulator activity score. Finally, examine cluster-specific activity patterns — e.g. Ptf1a elevated in acinar/early transitioning cells despite low expression, Sox9 increasing along acinar-to-ductal metaplasia, and differential Fosl2/AP1, Xbp1, and Dnajc21 activity between duct-like clusters.

## Related tools

- **scKINETICS** (Provides the fitted GRN matrix and dynamical model used to zero TF columns, recompute perturbed per-cell velocities, and run the TF ablation experiment) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment for the demonstration workflow including the TF ablation experiment and visualization of activity patterns) — http://github.com/dpeerlab/scKINETICS

## Examples

```
A_ko = A.copy(); A_ko[:, tf_idx] = 0; v_ko = sk.compute_velocity(A_ko, X); score[cell] = cosine(v_orig[cell], v_ko[cell])
```

## Evaluation signals

- Output tf_activity_scores.csv contains entries for all 585 TFs across all 5501 cells (585 x 5501 completeness)
- Ptf1a activity is elevated across acinar and early transitioning cell populations with declining regulatory strength along acinar-to-ductal metaplasia, despite its low expression in the data
- Sox9 activity increases progressively along the acinar-to-ductal metaplasia trajectory
- Fosl2/AP1 family factors, Xbp1, and Dnajc21 show distinct differential activity between the two duct-like clusters, reproducing the reported transdifferentiation regulatory landscape
- Each ablated TF's recomputed velocities differ from the original only through that TF's column removal in A (perturbation isolation check)

## Limitations

- The activity score reflects regulatory influence inferred from velocity perturbation under the learned GRN, not TF expression level — factors with low expression (like Ptf1a) can still score high
- Interpretation depends on the quality of the jointly inferred GRN and velocities, which rely on priors from epigenetic data, gene-gene coexpression, and phenotypic manifold constraints
- Each perturbation ablates one TF at a time; combinatorial regulator effects are not captured by the single-column zeroing scheme

## Evidence

- [abstract] scKINETICS jointly learns per-cell transcriptional velocities and a governing gene regulatory network via expectation-maximization: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [readme] README documents the TF ablation experiment as a core package functionality: [evidence span withheld — anchor status: external_doc]
- [intro] EM approach learns per-regulator impact on target genes, the quantity probed by column ablation: "an expectation-maximization approach derived to learn the impact of each regulator on its target genes"
