---
name: cosine-similarity-decomposition-evaluation
description: Use when a spot-level aggregated expression profile has been decomposed
  into per-cell expressions (e.g., via annealed Langevin posterior sampling) and you
  must quantify reconstruction accuracy against simulated ground truth.
when_to_use_negative:
- Input is real ST data without simulated ground-truth per-cell expressions — no reference
  exists against which cosine similarity can be computed
- Evaluating cell-type identification or nucleus segmentation steps rather than the
  gene expression decomposition step
- Comparing methods that only estimate cell-type proportions per spot (e.g. RCTD,
  Cell2location, CARD) since they do not output per-cell expressions
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SpatialScope
  role: Provides the score-based generative model and Langevin dynamics posterior
    sampling whose decomposed per-cell expressions are evaluated by cosine similarity
  repo: https://github.com/YangLabHKUST/SpatialScope
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:7fa8f96f16637ff1ea41407a6165d0058c72fa8b6ae525ec72f1085715b2a817
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - SpatialScope
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Cosine-similarity decomposition evaluation

## Summary

Validates SpatialScope's per-cell gene expression decomposition by comparing posterior-mean decomposed expressions of simulated multi-cell spots to known ground-truth per-cell expressions using cosine similarity, the metric the paper uses to benchmark decomposition performance.

## When to use

Use when a spot-level aggregated expression profile has been decomposed into per-cell expressions (e.g., via annealed Langevin posterior sampling) and you must quantify reconstruction accuracy against simulated ground truth.

## When NOT to use

- Input is real ST data without simulated ground-truth per-cell expressions — no reference exists against which cosine similarity can be computed
- Evaluating cell-type identification or nucleus segmentation steps rather than the gene expression decomposition step
- Comparing methods that only estimate cell-type proportions per spot (e.g. RCTD, Cell2location, CARD) since they do not output per-cell expressions

## Inputs

- Simulated spot-level aggregated expression profile y = x1 + x2 + noise with known per-cell ground truths and cell type labels (from scRNA-seq reference such as Ref_Heart_sanger_D2.h5ad)
- Posterior-mean per-cell decomposed expressions x1, x2 from annealed Langevin dynamics sampling
- Trained conditional score network checkpoint (e.g. model_5000.pt)

## Outputs

- Cosine similarity metrics between posterior-mean decomposed expressions and simulated ground-truth per-cell expressions
- decomposition_validation.csv containing the cosine-similarity metrics

## How to apply

Construct a simulated two-cell spot y = x1 + x2 + noise using the forward model y_i | X_i ~ N(f(X_i), sigma_y^2 I) with f = sum_m (exp(x_i,m + mu_k_i,m) - 1), so that per-cell expressions and cell type labels are known. Run annealed Langevin dynamics (L = 232 noise levels, T = 5 steps per level, eta = eta0 * sigma_l^2 / sigma_1^2) over R repeats, averaging final posterior samples to obtain posterior-mean expressions x1, x2. Compute cosine similarity between each posterior-mean per-cell expression and its simulated ground-truth cell, then write the metrics to decomposition_validation.csv. High cosine similarity to ground truth is the validation criterion, and the trained score-network checkpoint (e.g. model_5000.pt trained with sigma_begin=50, sigma_end=0.002) must load for posterior sampling.

## Related tools

- **SpatialScope** (Provides the score-based generative model and Langevin dynamics posterior sampling whose decomposed per-cell expressions are evaluated by cosine similarity) — https://github.com/YangLabHKUST/SpatialScope

## Evaluation signals

- Cosine similarity between posterior-mean decomposed expressions x1, x2 and simulated ground-truth cells is high (the paper's benchmark metric for decomposition)
- Trained score-network checkpoint (e.g. model_5000.pt) loads successfully for posterior sampling
- Metrics are written to decomposition_validation.csv with one cosine-similarity value per simulated cell
- Posterior means are obtained by averaging final Langevin samples over R repeats and L = 232 annealed noise levels
- Simulated spot uses the correct forward model with batch-corrected y_i (counts divided by exp(gamma_g_hat) and normalized to the reference mean per-cell total)

## Limitations

- Validation relies on simulated two-cell spots with known ground truth; accuracy on real spots with unknown per-cell truth is not directly measurable
- Training the deep generative models is time-consuming (measured in hours or days), so re-running the full sampling-evaluation loop is costly
- Results would not be satisfactory if platform effects between ST and single-cell reference data are not appropriately corrected before decomposition

## Evidence

- [methods] Compute cosine similarity between the posterior-mean decomposed expressions x1, x2 and the ground-truth simulated cells, and write the metrics to decomposition_validation.csv.: [evidence span withheld — anchor status: not_found]
- [methods] Sampling-based decomposition via posterior Langevin dynamics.: [evidence span withheld — anchor status: not_found]
- [methods] Decomposition is the third step of the SpatialScope pipeline being evaluated.: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [methods] Forward-model construction of simulated two-cell spots used as evaluation ground truth.: [evidence span withheld — anchor status: not_found]
