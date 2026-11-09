---
name: umap-embedding-of-single-cell-data
description: Use when two or more a priori class labels capture distinct biological
  sources of variance in single-cell data and you need one unsupervised embedding
  that preserves all of them simultaneously.
when_to_use_negative:
- Class labels are unknown or an unbiased view of the data is preferable — supervised
  LD axes are inappropriate in those situations
- More than 100 dimensions need feature selection — HSS is not feasible at that dimensionality
- Large heterogeneity exists within a single class or there are many populations —
  HSS-LDA performance suffers in these regimes
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: hsslda
  role: R package implementing runHSS (hybrid subset selection + LDA) and makeAxes
    used to generate the HSS-LD axes fed into UMAP
  repo: https://github.com/mamouzgar/hsslda
- name: MASS
  role: Alternative for running LDA without HSS; its lda()$scaling output feeds makeAxes
- name: dplyr
  role: Used to merge HSS-LD columns from the two per-label runHSS results into one
    combined feature dataframe
- name: ggplot2
  role: Renders the final biaxial plot of the embedded cells colored by class label
- name: UMAP
  role: Unsupervised dimensionality-reduction method applied to the merged HSS-LD
    feature set to produce the single embedding
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:49f2846e383e2afe86ab1336caeda10d7d136ce9a781b5aed2a8d8469e94a5dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mamouzgar/hsslda
  tools:
  - MASS
  - R
  - UMAP
  - dplyr
  - ggplot2
  - hsslda
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# umap-embedding-of-single-cell-data

## Summary

Generates a UMAP embedding of single-cell data using supervised HSS-LD axes as the input feature space, enabling a single biaxial plot that preserves multiple known sources of variance (e.g., cell line and cell-cycle phase from the Chromotyping CyTOF dataset).

## When to use

Use when two or more a priori class labels capture distinct biological sources of variance in single-cell data and you need one unsupervised embedding that preserves all of them simultaneously.

## When NOT to use

- Class labels are unknown or an unbiased view of the data is preferable — supervised LD axes are inappropriate in those situations
- More than 100 dimensions need feature selection — HSS is not feasible at that dimensionality
- Large heterogeneity exists within a single class or there are many populations — HSS-LDA performance suffers in these regimes

## Inputs

- CyTOF expression matrix (cells as rows, markers as columns), e.g. the Chromotyping dataset
- Two label vectors for the same cells (e.g., cell-cycle phase: G0/G1/S/G2, and cell line)
- HSS-LD feature dataframe merged from separate runHSS results (one per label)

## Outputs

- UMAP embedding of the combined HSS-LD feature space
- Biaxial plot (e.g., figure3h_biaxial.png) showing cell-line separation and preserved cell-cycle trajectory
- Per-model HSS-LDA objects with HSSscores, ElbowPlot, HSS-LDA-model, and HSS-LDA-result tables

## How to apply

Run hsslda::runHSS separately for each class label (e.g., cell-cycle phase and cell line on the Chromotyping CyTOF expression matrix) with score.method = 'euclidean', using the returned ElbowPlot to identify the optimal feature set for each model. Project cells onto each model's LD axes with makeAxes(df = data, co = model$scaling), then merge the HSS-LD columns from both runs into a single combined feature dataframe with dplyr. Embed this merged HSS-LD feature set with UMAP rather than the raw marker space, because the LD axes are optimized linear combinations that separate each a priori class. Render the biaxial plot with ggplot2 (e.g., HSS_LD1 vs HSS_LD2 colored by label) and save as figure3h_biaxial.png. Correct application is judged by expert visual review: cell lines must cluster separately while a G0-to-G2 cell-cycle trajectory is preserved within each cell line.

## Related tools

- **hsslda** (R package implementing runHSS (hybrid subset selection + LDA) and makeAxes used to generate the HSS-LD axes fed into UMAP) — https://github.com/mamouzgar/hsslda
- **MASS** (Alternative for running LDA without HSS; its lda()$scaling output feeds makeAxes)
- **dplyr** (Used to merge HSS-LD columns from the two per-label runHSS results into one combined feature dataframe)
- **ggplot2** (Renders the final biaxial plot of the embedded cells colored by class label)
- **UMAP** (Unsupervised dimensionality-reduction method applied to the merged HSS-LD feature set to produce the single embedding)

## Examples

```
hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE); lda.df = makeAxes(df = train.x, co = hss.result$`HSS-LDA-model`$scaling)
```

## Evaluation signals

- Visually confirm on the biaxial plot that cells cluster by cell line (expert review of the rendered figure)
- Confirm a G0-to-G2 cell-cycle trajectory is preserved within each cell line in the embedding
- Check each runHSS ElbowPlot to verify the elbow point identifies the optimal feature set before projecting axes
- Verify the merged feature dataframe contains HSS-LD columns from both per-label models and that row order matches the original cells

## Limitations

- LDA maximizes class separability at the expense of local data structure, which affects downstream embeddings
- HSS enriches for features separating multiple classes and may disregard features only useful for a single class; relevant colinear features may not all be selected
- HSS-LDA does not capture non-linear relationships and may perform poorly with large within-class heterogeneity or an increasing number of populations
- Features selected by HSS require follow-up analysis to establish the nature and statistical significance of differences

## Evidence

- [results] Combining HSS-LDs from separate HSS-LDA analyses of cell line and cell cycle as a feature set input to UMAP yields a biaxial plot that preserves both cell line and cell-cycle relationships: [evidence span withheld — anchor status: not_found]
- [methods] runHSS signature: "You can run hybrid subset selection on a dataset using runHSS()."
- [methods] makeAxes projection: "You can use this final model to apply the same dimensionality reduction to new data using makeAxes."
- [methods] elbow plot for optimal feature set: "ElbowPlot: visualizes the elbow plot and calculated elbow point for the optimal feature set."
- [intro] benchmark against UMAP: "We compare and benchmark HSS optimized LDA against PCA, UMAP, and PHATE across three mass cytometry datasets"
- [discussion] supervised method inappropriate without labels: "This supervised approach is inappropriate in situations in which labels are unknown, or in which an unbiased view of the data is preferable."
- [readme] makeAxes call in README: "lda.df.newData = makeAxes(df = newdata, co =hss.result$`HSS-LDA-model`$scaling)"
