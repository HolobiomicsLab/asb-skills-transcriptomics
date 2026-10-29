---
name: supervised-dimensionality-reduction-lda
description: Use when single-cell data (cells-by-markers matrix) has known a priori
  class labels (e.g., day0-day5 time points, cell-cycle phase, cell line) and the
  goal is a low-dimensional, interpretable visualization or projection of unlabeled
  cells onto axes trained on labeled cells.
when_to_use_negative:
- Class labels are unknown, or an unbiased view of the data is preferable (per the
  article's discussion, the supervised approach is inappropriate in these situations)
- The dataset has more than 100 dimensions, where HSS is not feasible (plain LDA or
  sda may still be used)
- A large degree of heterogeneity exists within a given class (e.g., PBMCs labeled
  by time point) or the number of populations is large, where HSS-LDA performance
  suffers
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0092
tools:
- name: hsslda
  role: R package implementing LDA with hybrid subset selection; provides runHSS()
    and makeAxes() for feature selection, model fitting, and projection of new data
  repo: https://github.com/mamouzgar/hsslda
- name: MASS
  role: Fits the LDA model (MASS::lda) with or without HSS feature selection; scaling
    coefficients feed makeAxes to generate LD axes
- name: ggplot2
  role: Renders biaxial LD plots (HSS_LD1 vs HSS_LD2 colored by labels), elbow plots,
    and cyclical cell-cycle embeddings
- name: dplyr
  role: Merges markers, class labels, LD axes, and HSS-LD columns from separate runs
    (e.g., cell-cycle and cell-line models) into combined feature dataframes
- name: UMAP
  role: Downstream unsupervised embedding of merged HSS-LD feature sets from multiple
    HSS-LDA analyses (e.g., dual-label Chromotyping Figure 3H)
provenance:
  source_task_ids:
  - task_007
  - task_003
  - task_005
  - task_002
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2022.100536
    title: Supervised dimensionality reduction for exploration of single-cell data
      by HSS-LDA
  source_content_hash: sha256:69c0fbefaa976f71163f95fb210f2cb0aa8ddab3451f4a3b3a83f56d28603957
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
  - stats
  - tidyr
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# supervised-dimensionality-reduction-lda

## Summary

Repurposes linear discriminant analysis (LDA), with optional hybrid subset selection (HSS) feature selection, as a supervised dimensionality-reduction method to organize single-cell data (CyTOF, scRNA-seq) on interpretable LD axes that separate a priori class labels such as cell type, cell-cycle phase, or experimental time point.

## When to use

Use when single-cell data (cells-by-markers matrix) has known a priori class labels (e.g., day0-day5 time points, cell-cycle phase, cell line) and the goal is a low-dimensional, interpretable visualization or projection of unlabeled cells onto axes trained on labeled cells.

## When NOT to use

- Class labels are unknown, or an unbiased view of the data is preferable (per the article's discussion, the supervised approach is inappropriate in these situations)
- The dataset has more than 100 dimensions, where HSS is not feasible (plain LDA or sda may still be used)
- A large degree of heterogeneity exists within a given class (e.g., PBMCs labeled by time point) or the number of populations is large, where HSS-LDA performance suffers

## Inputs

- cells-by-markers dataframe (x), e.g., TcellHartmann2020_sampleData with 15 metabolic channels (GLUT1 through HIF1A)
- vector of a priori class labels (y), e.g., day0-day5, cell-cycle phase, or cell line
- separation-metric choice (score.method: 'euclidean', 'silhouette', 'pixel.entropy', 'pixel.density', or 'custom')
- optional custom scoring function for custom.score.method
- per-cell cell-cycle score matrix (G1.S, S, G2, G2.M, M.G1) for cyclical embedding tasks

## Outputs

- hss.result object with HSSscores, ElbowPlot, HSS-LDA-model, and HSS-LDA-result tables
- LD-axis coordinates (LD1-LD5 or HSS_LD1/HSS_LD2) for all input cells, including projected unseen data
- ggplot2 biaxial LD plots colored by class label (e.g., figures analogous to Figures 2G-2K, 3H, 6E)
- selected feature set at the elbow point (e.g., 13/17 Morphometry, 8/32 T cell metabolic regulome, 24/32 Chromotyping)
- CSV exports of per-cell scores, LD coordinates, and cross-validation metrics (e.g., cell_cycle_lda_results.csv)

## How to apply

Format the input as a cells-by-markers dataframe (x) plus a class-label vector (y); for CyTOF, up to ~32 markers works, and HSS is not feasible above 100 dimensions. Run hybrid subset selection via hsslda::runHSS(x, y, score.method = 'euclidean', downsample = FALSE), which performs forward plus reverse stepwise selection around LDA and identifies the optimal marker set at the elbow of the returned ElbowPlot (e.g., 10 of 15 metabolic markers in the TcellHartmann2020_sampleData vignette, with LD1 explaining 0.7906 of the trace). Alternatively, fit a plain LDA baseline with MASS::lda and skip feature selection. Project cells — including unseen data such as CD57high cells projected onto CD57low-trained axes — onto the LD axes using makeAxes(df, co = model$scaling), then visualize HSS_LD1 vs HSS_LD2 with ggplot2 colored by labels. The separation metric can be swapped ('euclidean', 'silhouette', 'pixel.entropy', 'pixel.density', or a custom function passed via custom.score.method), and HSS-LD axes from separate runs (e.g., cell-cycle and cell-line labels on the Chromotyping dataset) can be merged and fed into UMAP to capture multiple variance sources.

## Related tools

- **hsslda** (R package implementing LDA with hybrid subset selection; provides runHSS() and makeAxes() for feature selection, model fitting, and projection of new data) — https://github.com/mamouzgar/hsslda
- **MASS** (Fits the LDA model (MASS::lda) with or without HSS feature selection; scaling coefficients feed makeAxes to generate LD axes)
- **ggplot2** (Renders biaxial LD plots (HSS_LD1 vs HSS_LD2 colored by labels), elbow plots, and cyclical cell-cycle embeddings)
- **dplyr** (Merges markers, class labels, LD axes, and HSS-LD columns from separate runs (e.g., cell-cycle and cell-line models) into combined feature dataframes)
- **UMAP** (Downstream unsupervised embedding of merged HSS-LD feature sets from multiple HSS-LDA analyses (e.g., dual-label Chromotyping Figure 3H))

## Examples

```
hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE); lda.df.newData = makeAxes(df = newdata, co = hss.result$`HSS-LDA-model`$scaling)
```

## Evaluation signals

- The fitted model object exists and the HSS-LDA-result table contains all input markers, class labels, and LD axes (LD1-LD5), with LD1 explaining 0.7906 of the trace in the vignette
- Retained feature counts match the reported elbow-optimal sets: 13 of 17 (Morphometry), 8 of 32 (T cell metabolic regulome), 24 of 32 (Chromotyping)
- HSS-LDA visualizations qualitatively retain or improve class-label separation compared with plain MASS::lda on the full marker set
- For projection tasks, unseen cells (e.g., CD57high day 5) overlap labeled training cells (CD57low days 2-3) on the LD manifold, as in Figures 2G-2K
- For the cyclical cell-cycle embedding, cross-validated phase-classification accuracy approximates ~88% with >99% of errors occurring between adjacent phases

## Limitations

- Relevant colinear features may not all be selected by HSS, since keeping multiple colinear features may not improve model separability
- HSS enriches for features useful for separating multiple classes and may disregard features only useful for separating a single class
- LDA maximizes class separability at the expense of local data structure and does not capture non-linear relationships
- Selected features are differentially expressed by one or more labeled classes; follow-up analysis is needed to establish the nature and statistical significance of the differences
- Performance suffers with increasing numbers of populations and with high heterogeneity within a class (e.g., PBMCs labeled by time point)

## Evidence

- [abstract] LDA identifies linear combinations of predictors that optimally separate a priori classes, enabling the study of specific aspects of cellular heterogeneity.: "LDA identifies linear combinations of predictors that optimally separate a priori classes, enabling the study of specific aspects of cellular heterogeneity."
- [methods] runHSS signature: "runHSS(x, y, score.method, custom.score.method, downsample)"
- [methods] makeAxes for new data: "You can use this final model to apply the same dimensionality reduction to new data using makeAxes."
- [discussion] supervised approach inappropriate without labels: "This supervised approach is inappropriate in situations in which labels are unknown, or in which an unbiased view of the data is preferable."
- [discussion] colinear features caveat: "Like other feature selection approaches, relevant colinear features may not all be selected by HSS as the separability of the model may not be improved by keeping multiple colinear features."
- [readme] README runHSS call: "hss.result = runHSS(x = train.x, y = train.y, score.method = 'euclidean', downsample = FALSE)"
