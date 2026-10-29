---
name: logistic-regression-roc-fate-prediction
description: Use when a LatentVelo model with a zr_dim regulatory state has been trained
  on lineage-deciding data (e.g., fibroblast reprogramming) and you need to quantitatively
  test whether the latent regulatory state separates outcome classes such as reprogrammed
  versus dead-end cells.
when_to_use_negative:
- When no per-cell outcome labels (e.g., reprogrammed/dead-end annotations) are available
  to train or evaluate the classifier — the ROC requires ground-truth fate labels
- When the trained LatentVelo model has no zr regulatory dimensions (zr_dim not set),
  since the classifier specifically tests the latent regulatory state
- For datasets with sparse cell clusters or disconnected populations where LatentVelo
  trajectory assumptions fail, as zr separation may be confounded by trajectory inference
  artifacts
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: LatentVelo
  role: Produces the latent regulatory state (zr) and latent AnnData that the logistic
    regression is trained on
  repo: https://github.com/Spencerfar/LatentVelo
- name: scikit-learn
  role: Fits the logistic regression on zr and computes the ROC curve for fate prediction
- name: scVelo
  role: Computes the velocity graph on the latent AnnData for streamline visualization
    and the CellRank single-terminal-state control
- name: CellRank
  role: Control analysis using scVelo velocities to confirm only a single (dead-end)
    terminal state is recovered, contrasting with zr-based fate separation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:3b1defe105c5c501cb4a272d20e297c21cd6a4142238eee988f09300ec423053
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Spencerfar/LatentVelo
  repo_url: https://github.com/Spencerfar/LatentVelo
  tools:
  - CellRank
  - LatentVelo
  - scVelo
  - scanpy
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# logistic-regression-roc-fate-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a logistic regression classifier on the LatentVelo latent regulatory state (zr) to predict cell fate outcome (reprogrammed vs. dead-end), and evaluates the discriminative power of zr with an ROC curve, as done for the Biddy et al. fibroblast reprogramming dataset (Figure 3K).

## When to use

Use when a LatentVelo model with a zr_dim regulatory state has been trained on lineage-deciding data (e.g., fibroblast reprogramming) and you need to quantitatively test whether the latent regulatory state separates outcome classes such as reprogrammed versus dead-end cells.

## When NOT to use

- When no per-cell outcome labels (e.g., reprogrammed/dead-end annotations) are available to train or evaluate the classifier — the ROC requires ground-truth fate labels
- When the trained LatentVelo model has no zr regulatory dimensions (zr_dim not set), since the classifier specifically tests the latent regulatory state
- For datasets with sparse cell clusters or disconnected populations where LatentVelo trajectory assumptions fail, as zr separation may be confounded by trajectory inference artifacts

## Inputs

- LatentVelo latent AnnData from ltv.output_results containing the latent regulatory state zr per cell
- Per-cell fate/outcome labels (e.g., reprogrammed vs. dead-end) from the fibroblast reprogramming dataset (Biddy et al.)
- Trained LatentVelo ltv.models.VAE model and AnnData prepared with ltv.utils.standard_clean_recipe

## Outputs

- Fitted logistic regression model mapping zr to reprogramming outcome
- ROC curve (and associated AUC) showing that zr predicts reprogrammed cells (Figure 3K analog)
- Latent UMAP plot colored by zr showing separation of reprogrammed vs. dead-end cells

## How to apply

After running the LatentVelo pipeline (ltv.utils.standard_clean_recipe with spliced_key='spliced' and unspliced_key='unspliced', ltv.models.VAE with zr_dim set, ltv.train, and ltv.output_results), extract the latent regulatory state zr for each cell from the latent AnnData on the latent UMAP space. Visualize zr on the latent UMAP to check qualitative separation of reprogrammed versus dead-end cells before fitting. Fit a logistic regression (sklearn) with zr as features and the observed reprogramming outcome as the label, then compute the ROC curve / AUC to demonstrate zr is predictive of the reprogrammed state. Cross-check with ltv.cell_trajectories to show latent trajectories diverge toward reprogrammed and dead-end fates, and with a CellRank control on scVelo velocities, which recovers only a single (dead-end) terminal state, confirming that the zr-based classifier captures fate structure beyond conventional velocity.

## Related tools

- **LatentVelo** (Produces the latent regulatory state (zr) and latent AnnData that the logistic regression is trained on) — https://github.com/Spencerfar/LatentVelo
- **scikit-learn** (Fits the logistic regression on zr and computes the ROC curve for fate prediction)
- **scVelo** (Computes the velocity graph on the latent AnnData for streamline visualization and the CellRank single-terminal-state control)
- **CellRank** (Control analysis using scVelo velocities to confirm only a single (dead-end) terminal state is recovered, contrasting with zr-based fate separation)

## Examples

```
from sklearn.linear_model import LogisticRegression; from sklearn.metrics import roc_auc_score; clf = LogisticRegression().fit(Zr, outcome); auc = roc_auc_score(outcome, clf.predict_proba(Zr)[:,1])
```

## Evaluation signals

- zr visualized on the latent UMAP qualitatively separates reprogrammed from dead-end cells (Figure 3 analog)
- The logistic-regression ROC curve shows zr is predictive of the reprogramming outcome (Figure 3K analog), with AUC well above chance (0.5)
- ltv.cell_trajectories diverging trajectories split toward reprogrammed and dead-end fates on the latent UMAP
- CellRank run on scVelo-inferred velocities recovers only a single (dead-end) terminal state, confirming the zr classifier adds fate information beyond standard velocity
- Validation set autoencoder and trajectory reconstruction losses from ltv.train converge, indicating the underlying latent space is well-trained before classification

## Limitations

- LatentVelo assumes a single initial latent state from which all cell trajectories start, and that each cell is reachable by a continuous trajectory; disconnected populations can distort the latent space used for classification
- On the dentate gyrus dataset LatentVelo incorrectly assigned the lowest latent times to the disconnected microglia cluster, so inferred trajectories incorrectly start from it — a similar artifact could bias zr-based fate separation
- For some datasets LatentVelo requires hyperparameter adjustment (larger latent state dimension, cell-type annotated AnnotVAE, or gene subsetting) before zr is meaningful; heuristics for choosing hyperparameters are not fully established

## Evidence
