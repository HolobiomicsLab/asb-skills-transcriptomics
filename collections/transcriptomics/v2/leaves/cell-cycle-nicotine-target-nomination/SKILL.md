---
name: cell-cycle-nicotine-target-nomination
description: Use when comparing a treated vs. control single-cell RNA-seq dataset
  (e.g., 10X Chromium hESC, ~12,280 cells) to find genes whose periodic/circular expression
  kinetics change between conditions.
when_to_use_negative:
- The biological process is non-periodic or linear (Cyclum targets circular manifolds;
  use linear trajectory methods like Monocle or Wanderlust instead)
- You only need cell-cycle phase labels from marker genes rather than ab initio circular-pattern
  inference — Cyclum deliberately relies on the circular manifold, not marker genes
- Your data is raw unnormalized counts — the log2(x+1), centered (mean=1), scaled
  (variance=1) preprocessing must be applied first per the Cyclum FAQ
edam_operation: http://edamontology.org/operation_2495
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Cyclum
  role: Sinusoidal autoencoder that infers circular pseudo-time and per-gene circular-pattern
    magnitude for treated and control conditions
  repo: https://github.com/KChen-lab/Cyclum
- name: cyclum.tuning.CyclumAutoTune
  role: Automatically selects the proper number of linear components to find the 'most
    circular' manifold before fitting
  repo: https://github.com/KChen-lab/Cyclum
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:91c9bd911ea69fe053c573fb4bebab5c069462b243d3d441766bf842ce103f5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/Cyclum
  tools:
  - Cyclum
  - cyclum.tuning.CyclumAutoTune
  edam_operation: http://edamontology.org/operation_2495
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cell-cycle-nicotine-target-nomination

## Summary

Nominate condition-responsive (e.g., nicotine target) genes by fitting Cyclum's sinusoidal autoencoder separately to treated and control scRNA-seq data, quantifying each gene's circular-pattern magnitude per condition, and ranking genes by the treated-minus-control difference.

## When to use

Use when comparing a treated vs. control single-cell RNA-seq dataset (e.g., 10X Chromium hESC, ~12,280 cells) to find genes whose periodic/circular expression kinetics change between conditions.

## When NOT to use

- The biological process is non-periodic or linear (Cyclum targets circular manifolds; use linear trajectory methods like Monocle or Wanderlust instead)
- You only need cell-cycle phase labels from marker genes rather than ab initio circular-pattern inference — Cyclum deliberately relies on the circular manifold, not marker genes
- Your data is raw unnormalized counts — the log2(x+1), centered (mean=1), scaled (variance=1) preprocessing must be applied first per the Cyclum FAQ

## Inputs

- Nicotine-treated 10X Chromium hESC count matrix (12,280 cells)
- Control 10X Chromium hESC count matrix
- Gene-by-cell expression matrices (transformed via log2(x+1), centered mean=1, scaled variance=1)

## Outputs

- Circular pseudo-time per cell for treated and control conditions
- Per-gene circular-pattern magnitude for each condition
- Ranked table of treated-minus-control circular-pattern magnitude differences (gene_circular_magnitude_ranking.csv)

## How to apply

Load the nicotine-treated and control 10X Chromium count matrices and transform each with log2(x+1), centering and scaling to variance 1 as recommended in the Cyclum FAQ. Fit the Cyclum sinusoidal autoencoder (cyclum.models) separately to each condition to infer circular pseudo-time, optionally using cyclum.tuning.CyclumAutoTune to select the optimal number of linear components for the 'most circular' manifold. With cyclum.evaluation, quantify each gene's circular-pattern magnitude — the degree of match between its expression kinetics and the inferred periodicity — in each condition. Compute the treated-minus-control difference per gene, sort in descending order, and check that known targets such as LDHA and ENO1 appear at the top, as reported in Supplementary Fig. 8b. Cyclum is a model-based approach fitting data to predefined circular manifolds, which confers robustness to random noise and small sample sizes; it can handle 12,280 cells x 33,694 genes in ~40 min on a laptop with a GTX 960M GPU.

## Related tools

- **Cyclum** (Sinusoidal autoencoder that infers circular pseudo-time and per-gene circular-pattern magnitude for treated and control conditions) — https://github.com/KChen-lab/Cyclum
- **cyclum.tuning.CyclumAutoTune** (Automatically selects the proper number of linear components to find the 'most circular' manifold before fitting) — https://github.com/KChen-lab/Cyclum

## Examples

```
from cyclum.tuning import CyclumAutoTune; tuner = CyclumAutoTune(); tuner.fit(train_df); model = tuner.show_history(); model.train(train_df, epoch=300, verbose=100)
```

## Evaluation signals

- LDHA and ENO1 appear among the top-ranked genes by treated-vs-control circular-pattern magnitude difference, reproducing the Supplementary Fig. 8b nomination
- The ranked difference table is sorted in descending order of treated-minus-control circular-pattern magnitude
- Fitted data lie on the circular manifold (CyclumAutoTune converges on the optimal number of linear components for each condition)
- Per-condition expression matrices pass the preprocessing invariant: log2(x+1) transformed, centered to mean=1, scaled to variance=1

## Limitations

- Runtime and hardware: analysis of 12,280 cells x 33,694 genes takes ~40 min and assumes GPU support (GTX 960M-class); CPU-only runs may be slower
- Cyclum is implemented in Python with Keras/TF 1.x; the README pins specific dependency versions (keras==2.2.4, tensorflow==1.14.0, numpy==1.16.5, etc.) and recommends forcing these if newer versions fail
- No version numbers for Python/Keras/TensorFlow are stated in the paper itself, so exact software provenance must be taken from the README's tested-versions table

## Evidence

- [intro] Cyclum can be used to identify genes associated with the periodic process, based on the degrees of match between the kinetics of gene expressions and the inferred periodicity.: "Cyclum can be used to identify genes associated with the periodic process, based on the degrees of match between the kinetics of gene expressions and the inferred periodicity."
- [methods] scaled (variance = 1), centered (mean = 1) log-transformed log2(x+1) data work well.: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [discussion] Cyclum can analyze an scRNA-seq dataset consisting of 12,280 cells 33,694 genes in 40 min on a laptop computer with a GTX 960M GPU and 2 GB graphic memory: "Cyclum can analyze an scRNA-seq dataset consisting of 12,280 cells 33,694 genes in 40 min on a laptop computer with a GTX 960M GPU and 2 GB graphic memory (detailed in Supplementary Note 2)."
- [readme] We also implemented class `cyclum.tuning.CyclumAutoTune`, which automatically select the proper number of linear components to help find the "most circular" manifold.: [evidence span withheld — anchor status: external_doc]
- [other] Nicotine target genes, including LDHA and ENO1, were nominated by evaluating the difference in circular pattern of each gene between the nicotine-treated sample and the control sample.: [evidence span withheld — anchor status: not_found]
