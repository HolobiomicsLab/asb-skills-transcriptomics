---
name: gaussian-likelihood-scoring-of-clusters
description: Use when spatial transcriptomics clusterings must be ranked without ground-truth
  labels, e.g. to pick the alpha/beta configuration by the goodness score v(y,Cα)
  instead of ARI.
when_to_use_negative:
- Ground-truth annotations exist and adjusted Rand index (ARI) can be computed directly,
  making the annotation-free proxy unnecessary
- No preprocessed PCs or spot coordinates are available, since both the Gaussian likelihood
  and the spatial Potts prior require them
- The clustering is not spatial (no spot neighborhood graph), so the Potts-model term
  is undefined
tools:
- name: ScribbleDom
  role: Pipeline whose clustering outputs are ranked by the goodness score to select
    optimal hyperparameter values
  repo: https://github.com/1alnoman/ScribbleDom
- name: Mclust
  role: Produces the automated scribble / initial cluster assignments whose goodness
    scores are evaluated
- name: Python 3.10.6
  role: Implementation environment in which the Gaussian likelihood, Potts prior,
    and variance penalty are computed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad594
    title: 'ScribbleDom: using scribble-annotated histology images to identify domains
      in spatial transcriptomics data'
  source_content_hash: sha256:aeff8f66fffc7d031ee425b8c1065a827cc5179c093d6aca774c8f8aae552fd5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1alnoman/ScribbleDom
  tools:
  - Mclust
  - Python
  - Python 3.10.6
  - ScribbleDom
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gaussian-likelihood-scoring-of-clusters

## Summary

Computes the ARI-independent goodness score v(y,Cα) for a spatial-transcriptomics clustering by combining a Gaussian likelihood of the spot PCs, a Potts-model prior on the neighborhood graph, and a quasi-likelihood variance penalty. This lets ScribbleDom select optimal hyperparameters (e.g. alpha/beta options) without manual annotations.

## When to use

Use when spatial transcriptomics clusterings must be ranked without ground-truth labels, e.g. to pick the best alpha/beta configuration by the goodness score v(y,Cα) instead of ARI.

## When NOT to use

- Ground-truth annotations exist and adjusted Rand index (ARI) can be computed directly, making the annotation-free proxy unnecessary
- No preprocessed PCs or spot coordinates are available, since both the Gaussian likelihood and the spatial Potts prior require them
- The clustering is not spatial (no spot neighborhood graph), so the Potts-model term is undefined

## Inputs

- Preprocessed spot principal components (n_pcs PCs computed from the top 2000 highly variable genes) in the preprocessed_data_folder
- Spot coordinates from the preprocessed data folder
- Clustering assignments for each evaluated configuration (e.g. from Mclust auto-scribble or ScribbleDom model output)
- config JSON specifying n_pcs, technology (visium/st), and hyperparameter grids such as alpha_options and beta_options

## Outputs

- goodness_scores.csv containing one finite numeric score per evaluated clustering configuration

## How to apply

Load the preprocessed spot principal components (n_pcs PCs from the top 2000 highly variable genes), spot coordinates, and the clustering assignments from the preprocessed data folder. Fit a Gaussian to each assigned cluster and evaluate each spot's PC vector under it to get the likelihood term. Compute the Potts-model prior over the spot neighborhood graph on the cluster assignments, using gamma=2 for ST technology and gamma=3 for Visium. Compute the within-cluster PC variance penalty with lambda=100, then combine all three into the product v(y,Cα)=q(y,Cα)·∏ p(yi|Ciα)·p(Ciα). Write one score per evaluated clustering configuration to goodness_scores.csv and select the clustering with the maximum score.

## Related tools

- **ScribbleDom** (Pipeline whose clustering outputs are ranked by the goodness score to select optimal hyperparameter values) — https://github.com/1alnoman/ScribbleDom
- **Mclust** (Produces the automated scribble / initial cluster assignments whose goodness scores are evaluated)
- **Python 3.10.6** (Implementation environment in which the Gaussian likelihood, Potts prior, and variance penalty are computed)

## Examples

```
chmod +x run_other_visium.sh && ./run_other_visium.sh config_expert.json config_mclust.json
```

## Evaluation signals

- goodness_scores.csv exists and contains a finite numeric score for each evaluated clustering configuration
- Exactly one score row per evaluated clustering configuration (e.g. each alpha/beta combination from the config grids)
- The selected optimal clustering equals the configuration with the maximum score in goodness_scores.csv
- Recomputing the score on the same clustering and preprocessed PCs reproduces the same value (deterministic given fixed gamma and lambda=100)
- The score correctly reflects the trade-off between Gaussian PC likelihood, Potts spatial smoothness (gamma=2 for ST, gamma=3 for Visium), and within-cluster PC dispersion

## Limitations

- The penalty weight (lambda=100) and Potts gamma values (2 for ST, 3 for Visium) are fixed constants from the paper and may need re-tuning for other technologies or spot layouts
- The score is an ARI-independent proxy; it ranks configurations but does not quantify agreement with true domains
- Requires the ScribbleDom preprocessing step (top 2000 highly variable genes, n_pcs PCs) to have been run first

## Evidence

- [full_text] Goodness score formula: [evidence span withheld — anchor status: not_found]
- [full_text] Inputs from preprocessing: [evidence span withheld — anchor status: not_found]
- [full_text] Technology-specific gamma: [evidence span withheld — anchor status: not_found]
- [readme] ARI-free hyperparameter selection: "This step will produce preprocessed data with principal components of spots, coordinates and mclust_backbone.csv as Automated Scribble in the folder"
