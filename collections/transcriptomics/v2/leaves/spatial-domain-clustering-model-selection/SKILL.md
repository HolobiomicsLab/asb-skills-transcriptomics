---
name: spatial-domain-clustering-model-selection
description: Use when multiple spatial-domain clusterings (e.g. across alpha_options
  0.05–0.95) must be ranked without ground-truth labels or manual ARI computation,
  to pick the optimal model configuration.
when_to_use_negative:
- Ground-truth domain annotations are available and ARI can be computed directly,
  making the annotation-free goodness score unnecessary
- The clustering output has already been fixed (no alpha_options or hyperparameter
  sweep to rank), so there is no model-selection decision to make
- Data has not been preprocessed yet (no PCs from top 2000 highly variable genes and
  no spot coordinates in preprocessed_data_folder)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ScribbleDom
  role: Pipeline whose hyperparameter sweep (alpha_options, beta_options, seed_options)
    produces the candidate clusterings ranked by the goodness score
  repo: https://github.com/1alnoman/ScribbleDom
- name: Mclust
  role: Generates the automated scribble / initial clustering assignments evaluated
    by the goodness score in the mclust schema
  repo: https://github.com/1alnoman/ScribbleDom
- name: Python
  role: Language used to compute the Gaussian likelihood, Potts-model prior (gamma=2
    ST / gamma=3 Visium), and lambda=100 variance penalty, writing goodness_scores.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad594
    title: 'ScribbleDom: using scribble-annotated histology images to identify domains
      in spatial transcriptomics data'
  source_content_hash: sha256:6917f02b1de3f3912f8618cfc35035ac15d01bde5e46737f8e516278b5d14d6b
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# spatial-domain-clustering-model-selection

## Summary

Computes ScribbleDom's ARI-independent goodness score v(y,Cα)=q(y,Cα)∏p(yi|Ciα)p(Ciα) over candidate clusterings so the best hyperparameter setting (e.g. alpha_options) can be selected without manual annotations.

## When to use

Use when multiple spatial-domain clusterings (e.g. across alpha_options 0.05–0.95) must be ranked without ground-truth labels or manual ARI computation, to pick the optimal model configuration.

## When NOT to use

- Ground-truth domain annotations are available and ARI can be computed directly, making the annotation-free goodness score unnecessary
- The clustering output has already been fixed (no alpha_options or hyperparameter sweep to rank), so there is no model-selection decision to make
- Data has not been preprocessed yet (no PCs from top 2000 highly variable genes and no spot coordinates in preprocessed_data_folder)

## Inputs

- Preprocessed spot principal components (n_pcs PCs derived from top 2000 highly variable genes) in preprocessed_data_folder/{dataset}/{samples[i]}/
- Spot coordinates from the preprocessed data folder
- Candidate clustering assignments y for each hyperparameter configuration (e.g. each alpha in alpha_options)
- Technology type ('visium' or 'st') from config JSON, governing the Potts gamma value (3 vs 2)

## Outputs

- goodness_scores.csv containing one finite numeric goodness score per evaluated clustering configuration
- A selected best-scoring clustering configuration (e.g. optimal alpha) for downstream final_outputs

## How to apply

Load the preprocessed spot principal components (n_pcs PCs from the top 2000 highly variable genes), spot coordinates, and each candidate clustering assignment from the preprocessed_data_folder. For each candidate configuration (e.g. each alpha in alpha_options [0.05,0.1,...,0.95]), compute three components: the Gaussian likelihood of each spot's PC vector under a Gaussian fitted to its assigned cluster; the Potts-model prior over the spot neighborhood graph using gamma=2 for ST technology and gamma=3 for Visium; and the quasi-likelihood penalty q(y,Cα) on within-cluster PC variance with lambda=100. Combine them as the product v(y,Cα)=q(y,Cα)∏i=1n p(yi|Ciα)p(Ciα) and write one score row per evaluated configuration to goodness_scores.csv; the highest-scoring clustering is the selected model.

## Related tools

- **ScribbleDom** (Pipeline whose hyperparameter sweep (alpha_options, beta_options, seed_options) produces the candidate clusterings ranked by the goodness score) — https://github.com/1alnoman/ScribbleDom
- **Mclust** (Generates the automated scribble / initial clustering assignments evaluated by the goodness score in the mclust schema) — https://github.com/1alnoman/ScribbleDom
- **Python** (Language used to compute the Gaussian likelihood, Potts-model prior (gamma=2 ST / gamma=3 Visium), and lambda=100 variance penalty, writing goodness_scores.csv)

## Examples

```
./run_other_visium.sh config_expert.json config_mclust.json
```

## Evaluation signals

- goodness_scores.csv exists and contains a finite numeric score for every evaluated clustering configuration
- goodness_scores.csv contains exactly one score row per evaluated configuration (e.g. 19 rows for the 19-value alpha_options sweep)
- The selected best-scoring clustering matches the configuration reported in final_outputs for the sample
- The Potts gamma parameter used matches the declared technology (gamma=2 for ST, gamma=3 for Visium) and the variance penalty uses lambda=100
- Score ordering changes meaningfully across alpha values; degenerate clusterings (e.g. single giant cluster) receive lower scores due to the within-cluster PC variance penalty

## Limitations

- Scores are ARI-independent proxies; they can rank clusterings differently from expert-annotation-based ARI, and competitiveness without expert input is only claimed empirically on DLPFC and melanoma samples
- The score depends on preprocessing choices (n_pcs, top 2000 highly variable genes); different PC counts change the likelihood and variance-penalty terms and hence the selected model

## Evidence

- [methods] goodness score as product of three components: [evidence span withheld — anchor status: not_found]
- [abstract] unsupervised operation without expert input: "Notably, when the expert input is absent, ScribbleDom can still operate, in a fully unsupervised manner like the state-of-the-art methods, and produces results that remain competitive."
- [readme] alpha sweep defines candidate clusterings to rank: ""alpha_options": [0.05,0.1,0.15,0.2,0.25,0.3,0.35,0.4,0.45,0.5,0.55,0.6,0.65,0.7,0.75,0.8,0.85,0.9,0.95],"
- [readme] preprocessing produces PCs and coordinates fed to scoring: "This step will produce preprocessed data with principal components of spots, coordinates and mclust_backbone.csv as Automated Scribble"
