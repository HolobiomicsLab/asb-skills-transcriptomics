---
name: cross-validation-parameter-selection
description: Use when fitting FateCompass's linear model of gene regulation on single-cell
  data and needing to control model complexity via the diffusion step t, which must
  be tuned by cross-validation rather than set heuristically.
when_to_use_negative:
- Working with bulk RNA-seq, where the original ISMARA linear model applies without
  the single-cell data-diffusion regularization FateCompass adds.
- The analysis goal is estimating transition probabilities from RNA velocity or differentiation
  potential — this skill only regularizes the TF-activity linear model step.
- No expressed-gene/binding-site matrix is available, since the promoter/gene set
  must be splittable 80/20 for cross-validation.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: FateCompass
  role: Provides the linear model of gene regulation whose data-diffusion regularization
    step t is tuned by cross-validation
  repo: https://github.com/sarajimenez/fatecompass
- name: Scanpy
  role: Used to load and pre-process the annotated single-cell adata object feeding
    the graph construction
- name: Python
  role: Implementation language; FateCompass is fully implemented in python (fatecompass.py
    module)
  repo: https://github.com/sarajimenez/fatecompass
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:b248696946865e3a5db73f149e574948bcfc965fa62e72cdacaed5f55aa067f6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sarajimenez/fatecompass
  repo_url: https://github.com/sarajimenez/fatecompass
  tools:
  - FateCompass
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-validation-parameter-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects the data-diffusion regularization parameter t in FateCompass's linear model of gene regulation by sweeping candidate diffusion-operator powers, fitting the regularized model on an 80% promoter/gene training split, and choosing the t that minimizes held-out test MSE.

## When to use

Use when fitting FateCompass's linear model of gene regulation on single-cell data and needing to control model complexity via the diffusion step t, which must be tuned by cross-validation rather than set heuristically.

## When NOT to use

- Working with bulk RNA-seq, where the original ISMARA linear model applies without the single-cell data-diffusion regularization FateCompass adds.
- The analysis goal is estimating transition probabilities from RNA velocity or differentiation potential — this skill only regularizes the TF-activity linear model step.
- No expressed-gene/binding-site matrix is available, since the promoter/gene set must be splittable 80/20 for cross-validation.

## Inputs

- Pre-processed mouse E15.5 adata object with cell-type ('clusters') annotations (Scanpy)
- TF binding-site matrix (pandas DataFrame; columns = TFs/motifs, rows = expressed genes)
- Candidate diffusion step values t for the sweep

## Outputs

- regularized_activity_matrix.csv (TF activity matrix refit at the selected t)
- mse_vs_t.png (test MSE vs. candidate t selection curve)
- Selected optimal diffusion step t (minimizer of held-out test MSE)

## How to apply

Build the cell-cell adjacency graph in gene-expression space from the pre-processed adata object, add self-loops, and row-normalize it. Implement the diffusion operator as the adjacency matrix raised to the tth power, so cells share information with local neighbors on the k-nearest-neighborhood graph and learned TF activities are smoothed before fitting the linear model (observed expression modeled as a linear combination of regulatory sites and TF activities from the TF binding-site matrix). Split the promoter/gene set 80/20 into training and test sets, fit the regularized linear model on the 80% split, and record test MSE for each candidate t. Sweep over candidate t values, select the t with minimum test MSE, refit on the full data at the selected t, and export the regularized TF activity matrix. The rationale is that the diffusion step t regularizes the ISMARA-style linear model for single-cell data, smoothing noisy per-cell activities; cross-validation over t replaces hand tuning of this pipeline parameter.

## Related tools

- **FateCompass** (Provides the linear model of gene regulation whose data-diffusion regularization step t is tuned by cross-validation) — https://github.com/sarajimenez/fatecompass
- **Scanpy** (Used to load and pre-process the annotated single-cell adata object feeding the graph construction)
- **Python** (Implementation language; FateCompass is fully implemented in python (fatecompass.py module)) — https://github.com/sarajimenez/fatecompass

## Examples

```
import scanpy as sc, pandas as pd; adata = sc.read_h5ad('E155_adata.h5ad'); motif_mat = pd.read_csv('binding_sites.csv', index_col=0); A = build_knn_adjacency(adata, self_loops=True, row_normalize=True); best_t = select_t_by_cv_mse(A, motif_mat, adata.to_df(), t_candidates=range(1,11), split=0.8)
```

## Evaluation signals

- The MSE-vs-t curve is monotonically interpretable and the chosen t is the arg-minimum of held-out test MSE on the 20% promoter/gene split.
- Refitting at the selected t on the full data yields a TF activity matrix matching the expected schema (rows = genes/regions context, columns = TFs/motifs) exported to regularized_activity_matrix.csv.
- Row-normalized adjacency with self-loops raises a valid stochastic matrix, so diffused values remain in a sane numeric range (no NaN/inf after raising to power t).
- TF activities at the selected t are smoother across neighboring cells on the kNN graph than the undiffused fit, consistent with the regularization's purpose.
- Identified activities remain stable under the 80/20 resplit (low variance in test MSE across candidate t near the optimum).

## Limitations

- Pipeline input parameters must be hand tuned; the article notes the pipeline depends on hand-tuned inputs, mitigated by detailed tutorials.
- TF activities use binding sites only in promoter regions (computationally predicted regulatory sites), which biases the gene-regulatory network structure; distal/long-range interactions are not considered.
- Trajectories/neighborhoods are built on a nearest-neighbor graph in reduced gene-expression space, and dimensionality-reduction choices (e.g., UMAP) lack biological interpretability, affecting the diffusion neighborhood.

## Evidence
