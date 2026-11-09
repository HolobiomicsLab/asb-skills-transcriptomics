---
name: within-cluster-variance-penalty-computation
description: Use when one or more candidate spatial-domain clusterings of ST/Visium
  spots (e.g., Mclust automated scribbles or ScribbleDom model outputs over an alpha_options
  grid) must be ranked by a single scalar score in the absence of ground-truth labels
  for ARI.
when_to_use_negative:
- Ground-truth domain annotations are available and adjusted Rand index can be computed
  directly — use ARI-based evaluation instead of this proxy score.
- The input is not a spatial spot-level clustering with coordinates (e.g., non-spatial
  single-cell clusters), since the Potts-model neighborhood term is undefined.
- Raw count matrices have not yet been preprocessed into PCs of the top 2000 highly
  variable genes — run the preprocessing step first.
edam_operation: http://edamontology.org/operation_3432
tools:
- name: ScribbleDom
  role: Framework whose goodness score v(y,Cα) is computed to rank candidate clusterings
    without ARI
  repo: https://github.com/1alnoman/ScribbleDom
- name: Mclust
  role: Produces the automated scribble clusterings whose outputs can be scored by
    this goodness measure
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad594
    title: 'ScribbleDom: using scribble-annotated histology images to identify domains
      in spatial transcriptomics data'
  source_content_hash: sha256:b56293e69d1cd9508966abebfb8513b37557daa9eb6165c5e371764832664f9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1alnoman/ScribbleDom
  tools:
  - Mclust
  - Python
  - ScribbleDom
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# within-cluster-variance-penalty-computation

## Summary

Computes ScribbleDom's ARI-independent goodness score v(y,Cα) = q(y,Cα)·∏ᵢ p(yᵢ|Cᵢα)·p(Cᵢα), which multiplies the Gaussian likelihood of spot PCs under their assigned cluster, a Potts-model prior over the spot neighborhood graph (gamma=2 for ST, gamma=3 for Visium), and a quasi-likelihood penalty q(y,Cα) on within-cluster PC variance (lambda=100), so optimal hyperparameters can be chosen without manual annotation.

## When to use

Use when one or more candidate spatial-domain clusterings of ST/Visium spots (e.g., Mclust automated scribbles or ScribbleDom model outputs over an alpha_options grid) must be ranked by a single scalar score in the absence of ground-truth labels for ARI.

## When NOT to use

- Ground-truth domain annotations are available and adjusted Rand index can be computed directly — use ARI-based evaluation instead of this proxy score.
- The input is not a spatial spot-level clustering with coordinates (e.g., non-spatial single-cell clusters), since the Potts-model neighborhood term is undefined.
- Raw count matrices have not yet been preprocessed into PCs of the top 2000 highly variable genes — run the preprocessing step first.

## Inputs

- Preprocessed spot principal components (n_pcs PCs from the top 2000 highly variable genes) from the preprocessed data folder
- Spot coordinates (spatial neighborhood structure for the Potts model)
- Clustering assignments to evaluate (e.g., mclust_backbone.csv automated scribble and/or ScribbleDom model outputs across alpha_options/seed_options)
- Technology label (visium or st) determining the Potts gamma value

## Outputs

- goodness_scores.csv with one finite numeric score per evaluated clustering configuration, enabling selection of the best-scoring clustering

## How to apply

Load the preprocessed spot principal components (n_pcs PCs computed from the top 2000 highly variable genes), the spot coordinates, and each clustering assignment to be evaluated. For each clustering, fit a Gaussian to the PC vectors of each assigned cluster and compute the per-spot likelihood p(yᵢ|Cᵢα); compute the Potts-model prior over the spot neighborhood graph on the assignments, using gamma=2 for ST technology and gamma=3 for Visium; and compute the quasi-likelihood penalty q(y,Cα) with lambda=100, which penalizes clusters whose spots show high PC dispersion. Multiply the three components into the goodness score v(y,Cα) and write one finite score row per evaluated configuration to goodness_scores.csv. The rationale is that this score stands in for the adjusted Rand index, letting ScribbleDom select its best alpha/seed configuration in a fully unsupervised manner when expert scribbles or annotations are absent.

## Related tools

- **ScribbleDom** (Framework whose goodness score v(y,Cα) is computed to rank candidate clusterings without ARI) — https://github.com/1alnoman/ScribbleDom
- **Mclust** (Produces the automated scribble clusterings whose outputs can be scored by this goodness measure)

## Evaluation signals

- goodness_scores.csv exists and contains a finite numeric score for every evaluated clustering configuration
- One score row per evaluated configuration so the argmax configuration can be unambiguously selected
- Score ordering is consistent with the penalty term's direction: clusterings with higher within-cluster PC dispersion receive lower scores under lambda=100
- The clustering with the maximum goodness score matches the configuration ScribbleDom emits as its final output
- Gamma is applied per technology: 2 for ST runs and 3 for Visium runs

## Limitations

- The score is an ARI-independent proxy; it may not perfectly track adjusted Rand index performance on all samples (ScribbleDom improved ARI on 9 of 12 DLPFC samples, not all 12).
- Fixed constants (gamma=2/3, lambda=100) were tuned for ST/Visium spot data and may not transfer to other spatial platforms or resolutions.
- Requires the exact preprocessed representation (PCs from the top 2000 highly variable genes, e.g. n_pcs=15 in the example config); scores are not comparable across different PC settings.

## Evidence

- [methods] Preprocessing: top 2000 HVGs and PCs: "Finds the top 2000 highly variable genes and calculates principal components from raw Gene expression data."
- [abstract] Unsupervised operation rationale: "Notably, when the expert input is absent, ScribbleDom can still operate, in a fully unsupervised manner like the state-of-the-art methods, and produces results that remain competitive."
- [readme] Hyperparameter grid being ranked by the score: ""alpha_options": [0.05,0.1,0.15,0.2,0.25,0.3,0.35,0.4,0.45,0.5,0.55,0.6,0.65,0.7,0.75,0.8,0.85,0.9,0.95],"
