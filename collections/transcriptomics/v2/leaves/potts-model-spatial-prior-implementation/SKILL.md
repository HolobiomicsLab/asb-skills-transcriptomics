---
name: potts-model-spatial-prior-implementation
description: Use when a clustering of spatial transcriptomics spots must be scored/compared
  without ground-truth labels — e.g. sweeping alpha_options/beta_options/seed_options
  in ScribbleDom configs to pick the alpha without computing ARI.
when_to_use_negative:
- Ground-truth domain annotations are available and ARI can be computed directly —
  the point of v(y,Cα) is to avoid requiring manual annotations like ARI.
- Input is raw gene expression rather than preprocessed data — this score operates
  on the preprocessed PC matrix, not the count matrix; run the R preprocessing step
  (top 2000 HVGs, PCs) first.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: ScribbleDom
  role: Pipeline in which the goodness score is computed to select optimal hyperparameter
    values without ARI
  repo: https://github.com/1alnoman/ScribbleDom
- name: Python
  role: Implementation language for computing the likelihood, Potts prior, and variance
    penalty terms (Python 3.10.6 recommended)
- name: Mclust
  role: Generates the automated scribble / candidate cluster assignments that the
    goodness score evaluates
  repo: https://github.com/1alnoman/ScribbleDom
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad594
    title: 'ScribbleDom: using scribble-annotated histology images to identify domains
      in spatial transcriptomics data'
  source_content_hash: sha256:509b98d3582abc4ca3b433ac65b2287502cf7687a30813e674b4723242b0c1e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1alnoman/ScribbleDom
  tools:
  - Mclust
  - Python
  - ScribbleDom
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# potts-model-spatial-prior-implementation

## Summary

Implements the ARI-independent goodness score v(y,Cα) that ScribbleDom uses to select clustering hyperparameters without manual annotation: the product of a Gaussian likelihood of spot PCs given cluster assignments, a Potts-model prior over the spot neighborhood graph, and a quasi-likelihood variance penalty q(y,Cα).

## When to use

Use when a clustering of spatial transcriptomics spots must be scored/compared without ground-truth labels — e.g. sweeping alpha_options/beta_options/seed_options in ScribbleDom configs to pick the best alpha without computing ARI.

## When NOT to use

- Ground-truth domain annotations are available and ARI can be computed directly — the point of v(y,Cα) is to avoid requiring manual annotations like ARI.
- Input is raw gene expression rather than preprocessed data — this score operates on the preprocessed PC matrix, not the count matrix; run the R preprocessing step (top 2000 HVGs, PCs) first.

## Inputs

- preprocessed spot principal components (n_pcs PCs derived from the top 2000 highly variable genes)
- spot spatial coordinates from the space ranger output
- candidate cluster assignments for each evaluated (alpha, beta, seed) configuration
- technology flag ('visium' or 'st') from the config JSON, which sets the Potts gamma

## Outputs

- goodness_scores.csv with one finite numeric score per evaluated clustering configuration

## How to apply

Load the preprocessed spot principal components (n_pcs PCs from the top 2000 highly variable genes), spot coordinates, and the candidate cluster assignments from the preprocessed data folder. Compute the Gaussian likelihood term, fitting a Gaussian to each cluster's PC vectors and evaluating each spot's PC vector under its assigned cluster. Compute the Potts-model prior over the spot neighborhood graph on the assignments, using gamma=2 for ST technology and gamma=3 for Visium. Compute the within-cluster PC variance penalty q(y,Cα) with lambda=100, which penalizes clusters whose spots have high PC dispersion. Combine as v(y,Cα)=q(y,Cα)·∏_{i=1}^{n} p(yi|Ciα)·p(Ciα), writing one finite score per evaluated configuration to goodness_scores.csv so the best-scoring clustering can be selected.

## Related tools

- **ScribbleDom** (Pipeline in which the goodness score is computed to select optimal hyperparameter values without ARI) — https://github.com/1alnoman/ScribbleDom
- **Python** (Implementation language for computing the likelihood, Potts prior, and variance penalty terms (Python 3.10.6 recommended))
- **Mclust** (Generates the automated scribble / candidate cluster assignments that the goodness score evaluates) — https://github.com/1alnoman/ScribbleDom

## Examples

```
./run_other_visium.sh config_expert.json config_mclust.json
```

## Evaluation signals

- goodness_scores.csv exists and contains a finite numeric score for every evaluated clustering configuration
- one score row per evaluated configuration, so the best-scoring clustering can be unambiguously selected
- score is reproducible for a fixed (alpha, beta, seed) configuration given the same preprocessed PCs and coordinates
- score ordering is sensitive to hyperparameters: distinct alpha_options values yield distinct scores rather than identical constants
- computed v(y,Cα) decomposes into the three named components (Gaussian likelihood, Potts prior, q penalty), so each term is finite and separately inspectable

## Limitations

- Gamma values are technology-specific (2 for ST, 3 for Visium); using the wrong gamma for the platform will distort the spatial prior.
- The score is a proxy — it can operate fully unsupervised, but the paper notes that without expert input ScribbleDom results are competitive rather than guaranteed superior to ARI-selected settings.
- Requires the preprocessing step (top 2000 highly variable genes, n_pcs PCs, Mclust) to have run first; it cannot be computed directly on raw count matrices.

## Evidence

- [other] Goodness score formula and components: [evidence span withheld — anchor status: not_found]
- [abstract] Unsupervised operation rationale: "Notably, when the expert input is absent, ScribbleDom can still operate, in a fully unsupervised manner like the state-of-the-art methods, and produces results that remain competitive."
- [readme] Preprocessing inputs: HVGs and PCs: "Finds the top 2000 highly variable genes and calculates principal components from raw Gene expression data."
- [abstract] Loss integrating expression similarity and scribble adherence: "ScribbleDom incorporates a loss function that integrates two crucial components: similarity in gene expression profiles and adherence to the valuable input of a human annotator through scribbles on"
