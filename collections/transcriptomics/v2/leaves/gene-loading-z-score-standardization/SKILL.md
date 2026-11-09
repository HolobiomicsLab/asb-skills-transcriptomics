---
name: gene-loading-z-score-standardization
description: Use when you have gene loading (weight) vectors from an ICA decomposition
  of log-transformed tumor scRNA-seq expression and need comparable loading scales
  across components before downstream clustering or meta-component mapping.
when_to_use_negative:
- Input is not an ICA-derived gene loading matrix — e.g. a cell-by-gene expression
  matrix that has not been decomposed by fastICA
- The goal is batch integration across datasets rather than within-dataset component
  normalization — Harmony or scVI-style joint low-dimensional representations are
  the alternatives discussed
- You only need MetaTiME cell-state annotation of new data via the pretrained MeC
  annotator, which uses published pretrained meta-components rather than re-deriving
  and re-normalizing loadings
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MetaTiME
  role: Framework whose ICA meta-component discovery produces the gene loading vectors
    that are z-score normalized and skewness aligned; also provides loaddata/testdata
    helpers and the pretrained MeC space for downstream annotation
  repo: https://github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:0211d171bce513e647d08feccda6c95a54a98c96616ded1e0410f6c8bd50ebd1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - MetaTiME
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# gene-loading-z-score-standardization

## Summary

A normalization step from the MetaTiME pipeline that z-score-transforms the gene loading values of each independent component (IC) from fastICA decomposition so that gene representation scores are comparable across the 100 components and across datasets.

## When to use

Use when you have gene loading (weight) vectors from an ICA decomposition of log-transformed tumor scRNA-seq expression and need comparable loading scales across components before downstream clustering or meta-component mapping.

## When NOT to use

- Input is not an ICA-derived gene loading matrix — e.g. a cell-by-gene expression matrix that has not been decomposed by fastICA
- The goal is batch integration across datasets rather than within-dataset component normalization — Harmony or scVI-style joint low-dimensional representations are the alternatives discussed
- You only need MetaTiME cell-state annotation of new data via the pretrained MeC annotator, which uses published pretrained meta-components rather than re-deriving and re-normalizing loadings

## Inputs

- log-transformed tumor scRNA-seq expression matrix (genes x cells)
- raw gene loading (weight) vectors from fastICA decomposition with n_components=100

## Outputs

- z-scored, skewness-aligned gene loading matrix (genes x 100 ICs), e.g. ic_loadings.tsv
- per-component sign-flip assignments ensuring positively skewed loading distributions

## How to apply

After running fastICA on a log-transformed tumor scRNA-seq expression matrix to obtain n_components=100 maximally independent gene-expression components, z-score-transform the gene loading values within each component so component scales are directly comparable among datasets. Then check the skewness of each component's loading distribution and flip the component's sign so the skewness is positive, yielding consistently oriented transcriptional directions. Assemble the processed loadings into a genes x 100 ICs matrix (e.g. ic_loadings.tsv) for downstream clustering and projection onto the MetaTiME meta-component (MeC) space. The rationale, per MetaTiME, is that ICA components from a large-scale pan-cancer integration optimize information independence among components, and z-weight normalization plus skewness alignment make the normalized IC loadings reusable for annotating cell states in new TME scRNA-seq data.

## Related tools

- **MetaTiME** (Framework whose ICA meta-component discovery produces the gene loading vectors that are z-score normalized and skewness aligned; also provides loaddata/testdata helpers and the pretrained MeC space for downstream annotation) — https://github.com/yi-zhang/MetaTiME

## Examples

```
from sklearn.decomposition import FastICA; import scipy.stats; ica = FastICA(n_components=100); Z = ica.fit_transform(log_expr.T).T; Z = [(z - z.mean())/z.std() if scipy.stats.skew(z) > 0 else -((z - z.mean())/z.std()) for z in Z]
```

## Evaluation signals

- Output loading matrix (ic_loadings.tsv) exists and contains exactly 100 component columns
- Gene loading values of each component have mean ≈ 0 and standard deviation ≈ 1 after z-score transformation
- Loading distributions of components are predominantly positively skewed after sign alignment
- Resulting normalized IC loadings are on comparable scales, as required for downstream clustering and MeC-space projection

## Limitations

- Normalization and sign alignment are defined for ICA loadings of tumor microenvironment (TME) scRNA-seq data; transferability to other data modalities is not established in the article
- The article notes GR ChIP-seq was unavailable in relevant T cell states, illustrating that supporting external annotations for downstream meta-component interpretation may be incomplete
- The MetaTiME README states the repo is continuously being improved and more details will be updated, so pipeline specifics may change between versions

## Evidence

- [discussion] performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51.: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51."
- [readme] MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells: "MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment."
- [other] Run fastICA on the expression matrix to decompose it into n_components=100 maximally independent gene-expression components.: [evidence span withheld — anchor status: not_found]
- [other] Check the skewness of each component and flip its sign so the skewness is positive, yielding consistently oriented directions.: [evidence span withheld — anchor status: not_found]
