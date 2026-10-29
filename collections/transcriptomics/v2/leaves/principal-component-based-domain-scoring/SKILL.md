---
name: principal-component-based-domain-scoring
description: Use when evaluating multiple spatial-domain clustering outputs (e.g.,
  over alpha_options/beta_options/seed_options grids) and ground-truth labels are
  unavailable, so clustering quality must be ranked by a model-based score instead
  of ARI.
when_to_use_negative:
- Ground-truth annotations exist for the sample — use ARI directly rather than this
  proxy score
- Input is still a raw count matrix; the top-2000 HVG / PC preprocessing step (get_genex_data_from_10x_h5.R
  or get_genex_data_from_rds_ST_data.R) must run first
- Data is non-spatial (no spot neighborhood graph can be built), since the Potts-model
  prior term requires spatial coordinates
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ScribbleDom
  role: Pipeline whose hyperparameter sweep is scored by the goodness function; implements
    the likelihood, Potts prior, and variance penalty terms
  repo: https://github.com/1alnoman/ScribbleDom
- name: Python
  role: Runtime used to compute the goodness scores for each candidate clustering
- name: Mclust
  role: Generates the automated scribble / initial clustering (mclust_backbone.csv)
    that serves as one candidate clustering input
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad594
    title: 'ScribbleDom: using scribble-annotated histology images to identify domains
      in spatial transcriptomics data'
  source_content_hash: sha256:1fbcf7fb10be125c0760f4fdd13157d6274e1b293ba48af136e17624c16976cc
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# principal-component-based-domain-scoring

## Summary

Computes ScribbleDom's ARI-independent goodness score v(y,Cα)=q(y,Cα)∏p(yi|Ciα)p(Ciα) that scores each candidate spatial-domain clustering by combining Gaussian likelihood of gene-expression PCs, a Potts-model spatial prior, and a within-cluster PC variance penalty — enabling hyperparameter selection (alpha, beta, seed) without manual annotations.

## When to use

Use when evaluating multiple spatial-domain clustering outputs (e.g., over alpha_options/beta_options/seed_options grids) and ground-truth labels are unavailable, so clustering quality must be ranked by a model-based score instead of ARI.

## When NOT to use

- Ground-truth annotations exist for the sample — use ARI directly rather than this proxy score
- Input is still a raw count matrix; the top-2000 HVG / PC preprocessing step (get_genex_data_from_10x_h5.R or get_genex_data_from_rds_ST_data.R) must run first
- Data is non-spatial (no spot neighborhood graph can be built), since the Potts-model prior term requires spatial coordinates

## Inputs

- Preprocessed spot principal components (n_pcs PCs from top 2000 highly variable genes) in {preprocessed_data_folder}/{dataset}/{samples[i]}/
- Spot coordinates from the preprocessed data folder (from spatial/tissue_positions_list.csv via preprocessing)
- Clustering assignments (cluster label C for each spot) for every evaluated hyperparameter configuration (alpha, beta, seed)
- config_mclust.json / config_expert.json specifying n_pcs, alpha_options, beta_options, seed_options, and technology (visium/st)

## Outputs

- goodness_scores.csv with one finite numeric score per evaluated clustering configuration
- Best-scoring clustering (final spatial domain assignments) written to {final_output_folder}/{dataset}/{samples[i]}/

## How to apply

Load the preprocessed spot principal components (n_pcs PCs from the top 2000 highly variable genes), spot coordinates, and the clustering assignments for each hyperparameter configuration from the preprocessed_data folder. Compute the Gaussian likelihood of each spot's PC vector under a Gaussian fitted to its assigned cluster, and the Potts-model prior over the spot neighborhood graph on the cluster assignments with gamma=2 for ST technology and gamma=3 for Visium. Add the quasi-likelihood penalty q(y,Cα) on within-cluster PC variance with lambda=100, which penalizes clusters whose spots have high PC dispersion. Combine the components into a single score v(y,Cα)=q(y,Cα)∏i=1n p(yi|Ciα)p(Ciα) and write one finite score per evaluated clustering to goodness_scores.csv; the best-scoring clustering across alpha_options (0.05–0.95), beta_options (0.25–0.4), and seed_options is then selected as the final domain output.

## Related tools

- **ScribbleDom** (Pipeline whose hyperparameter sweep is scored by the goodness function; implements the likelihood, Potts prior, and variance penalty terms) — https://github.com/1alnoman/ScribbleDom
- **Python** (Runtime used to compute the goodness scores for each candidate clustering)
- **Mclust** (Generates the automated scribble / initial clustering (mclust_backbone.csv) that serves as one candidate clustering input)

## Examples

```
chmod +x run_other_visium.sh && ./run_other_visium.sh config_expert.json config_mclust.json
```

## Evaluation signals

- goodness_scores.csv exists, is finite/numeric, and contains one score row per evaluated clustering configuration (alpha × beta × seed combinations)
- The selected best-scoring clustering corresponds to a configuration within the configured alpha_options [0.05,0.1,...,0.95] and beta_options [0.25,0.3,0.35,0.4] grids
- Potts prior gamma is set to 2 for ST technology and 3 for Visium, and the variance penalty uses lambda=100
- The PC vectors scored match the n_pcs setting (e.g., n_pcs=15) computed from the top 2000 highly variable genes
- The chosen clustering can be re-scored reproducibly (deterministic torch.cudnn settings when run on GPU per README)

## Limitations

- The goodness score is a model-based proxy, not ARI; it depends on fixed hyperparameter choices (gamma=2 for ST vs gamma=3 for Visium, lambda=100) tuned for the paper's data
- Without manual_scribble.csv only AutoScribbleDom results are produced — the score ranks automated candidates but cannot incorporate expert scribble adherence
- Score validity depends on the preprocessing having produced principal components of spots, coordinates and mclust_backbone.csv in the expected folder structure

## Evidence

- [methods] goodness score definition: [evidence span withheld — anchor status: not_found]
- [methods] gamma technology dependence: [evidence span withheld — anchor status: not_found]
- [abstract] motivation for unsupervised scoring: "Notably, when the expert input is absent, ScribbleDom can still operate, in a fully unsupervised manner like the state-of-the-art methods, and produces results that remain competitive."
- [readme] preprocessing inputs to scoring: "Finds the top 2000 highly variable genes and calculates principal components from raw Gene expression data. Also runs ```Mclust``` algorithm to create a automated scribble."
- [readme] hyperparameter grid scored: ""alpha_options": [0.05,0.1,0.15,0.2,0.25,0.3,0.35,0.4,0.45,0.5,0.55,0.6,0.65,0.7,0.75,0.8,0.85,0.9,0.95],"
