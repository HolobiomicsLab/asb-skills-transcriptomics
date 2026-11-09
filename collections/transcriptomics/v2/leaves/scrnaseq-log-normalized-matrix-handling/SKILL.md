---
name: scrnaseq-log-normalized-matrix-handling
description: Use when feeding raw perturbation (Y) and control (Z) cell-by-gene count
  matrices into scDecouple or similar scCRISPR-seq pipelines that require log-normalized,
  HVG-subtracted matrices as input.
when_to_use_negative:
- Input is already log-normalized (e.g. Y.norm.log.sub / Z.norm.log.sub from a prior
  data_preprocessing run) — re-running normalization would double-transform the data.
- The downstream analysis operates on features other than gene expression (e.g. ATAC
  peaks or surface protein counts) where log-normalized cell-by-gene semantics do
  not apply.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDecouple
  role: R package whose data_preprocessing (step 1) performs log-normalization and
    highly variable gene detection on Y and Z; also provides the one-click run_scDecouple
    wrapper
  repo: https://github.com/MengQiuchen/scDecouple
- name: devtools
  role: Used to install the scDecouple package from GitHub before the normalization
    functions are available
  repo: https://github.com/MengQiuchen/scDecouple
- name: tidyverse
  role: Loaded alongside scDecouple for handling the cell-by-gene matrices during
    preprocessing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae011
    title: 'scDecouple: decoupling cellular response from infected proportion bias
      in scCRISPR-seq'
  source_content_hash: sha256:649e6433e820f34f2351c9b05bb8b84d2811f728623d9c0567929df1e1b81be2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MengQiuchen/scDecouple
  tools:
  - R
  - devtools
  - mixtools
  - scDecouple
  - tidyverse
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# scrnaseq-log-normalized-matrix-handling

## Summary

Preparing and passing scRNA-seq / scCRISPR-seq data as log-normalized cell-by-gene matrices (with highly variable gene subsets) so that downstream methods like scDecouple's PC selection and GMM-EM decoupling operate on correctly normalized inputs.

## When to use

Use when feeding raw perturbation (Y) and control (Z) cell-by-gene count matrices into scDecouple or similar scCRISPR-seq pipelines that require log-normalized, HVG-subtracted matrices as input.

## When NOT to use

- Input is already log-normalized (e.g. Y.norm.log.sub / Z.norm.log.sub from a prior data_preprocessing run) — re-running normalization would double-transform the data.
- The downstream analysis operates on features other than gene expression (e.g. ATAC peaks or surface protein counts) where log-normalized cell-by-gene semantics do not apply.

## Inputs

- Y: perturbation-group cell-by-gene matrix (raw counts)
- Z: control-group cell-by-gene matrix (raw counts)

## Outputs

- Y.norm.log.sub: log-normalized perturbation-group matrix subset to highly variable genes
- Z.norm.log.sub: log-normalized control-group matrix subset to highly variable genes
- genes.variable: highly variable gene list
- Z.pca.results: PCA object for Z containing $x (scores), $rotation (loadings), and $sdev

## How to apply

In R, load the cell-by-gene matrices Y (perturbation group) and Z (control group), then call data_preprocessing(Y, Z) as scDecouple's step 1; it performs log-normalization and highly variable gene detection, returning Y.norm.log.sub, Z.norm.log.sub, and Z.pca.results ($x scores, $rotation, $sdev). These log-normalized, HVG-subset matrices are the required input format for the subsequent pc_selection and scDecouple GMM-EM decoupling steps — raw counts must not be passed downstream. The one-click wrapper run_scDecouple(Y, Z) applies this normalization automatically. Log-normalization is performed before PCA on the control group Z, so the PCA space in which cluster proportions and cellular responses are later estimated is defined on normalized expression profiles.

## Related tools

- **scDecouple** (R package whose data_preprocessing (step 1) performs log-normalization and highly variable gene detection on Y and Z; also provides the one-click run_scDecouple wrapper) — https://github.com/MengQiuchen/scDecouple
- **devtools** (Used to install the scDecouple package from GitHub before the normalization functions are available) — https://github.com/MengQiuchen/scDecouple
- **tidyverse** (Loaded alongside scDecouple for handling the cell-by-gene matrices during preprocessing)

## Examples

```
res.step1 <- data_preprocessing(Y, Z)
```

## Evaluation signals

- data_preprocessing(Y, Z) returns objects named Y.norm.log.sub, Z.norm.log.sub, genes.variable and Z.pca.results; absence of any of these indicates the step did not complete.
- Z.pca.results must contain $x, $rotation and $sdev components since downstream pc_selection consumes Z.pca.sdev = res.step1$Z.pca.results$sdev.
- Y.norm.log.sub and Z.norm.log.sub must be subset to the same highly variable gene set (genes.variable) so perturbation and control matrices are column-aligned.
- Values in Y.norm.log.sub / Z.norm.log.sub should be log-scale (non-negative, compressed relative to raw counts), not raw integer counts.
- Running the one-click run_scDecouple(Y, Z) should reproduce the same normalized intermediate outputs as running data_preprocessing step-by-step on the same seed.

## Limitations

- The decoupling downstream assumes cell distributions fit Gaussian mixture models in PC space; this holds well in certain cell lines and tissues but may not hold in intricate systems or disease tissues such as tumors.
- The same cell clusters between NT and perturbation groups assumption can fail when some rare cell types are completely absent from infection.
- The similar cellular responses across cell clusters assumption limits scDecouple to cells with moderate heterogeneity, which constrains the data this preprocessing should feed.

## Evidence

- [methods] step1: preprocessing, to get log-normalized data and highly variable genes: "## step1: preprocessing, to get log-normalized data and highly variable genes"
- [methods] res.step1 <- data_preprocessing(Y, Z): "res.step1 <- data_preprocessing(Y, Z)"
- [readme] Y: The perturbation group, cell-by-gene matrix.: "1. `Y`: The perturbation group, cell-by-gene matrix."
- [readme] Z: The control group, cell-by-gene matrix.: "2. `Z`: The control group, cell-by-gene matrix."
- [discussion] The package comes equipped with an integrated one-click data preprocessing pipeline: "The package comes equipped with an integrated one-click data preprocessing pipeline"
- [methods] demo data loading of test_data: "# demo data
data(test_data)"
