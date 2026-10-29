---
name: component-sign-skewness-alignment
description: Use when decomposing a log-transformed tumor scRNA-seq expression matrix
  with ICA (n_components=100) and you need IC gene loadings on a comparable scale
  with consistently oriented (positively skewed) directions before downstream clustering.
when_to_use_negative:
- Input is already a normalized loading/signature table or annotated feature set —
  no decomposition or sign alignment is needed.
- Goal is annotating new TME cells using MetaTiME's pretrained meta-components (MetaTiME-Annotator
  projection), not re-learning components from raw data.
- Expression matrix is not log-transformed single-cell TME data (e.g., bulk RNA-seq),
  where the ICA-based MetaTiME decomposition does not apply.
tools:
- name: MetaTiME
  role: Python package (pip install metatime) providing the ICA-based decomposition
    pipeline, loaddata/testdata helpers, and the z-weight normalization and skewness
    alignment transforms applied to IC loadings
  repo: https://github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:ed9657edea95ff0be01a519ef2ec7928be5ffedeb89d66675d50ec9d2b1b55c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - MetaTiME
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# component-sign-skewness-alignment

## Summary

Normalizes independent component (IC) gene loadings from fastICA decomposition of tumor scRNA-seq expression matrices by z-weight transformation and sign flipping to positive skewness, so that the scales of gene representation scores are comparable among the ~100 components and across datasets.

## When to use

Use when decomposing a log-transformed tumor scRNA-seq expression matrix with ICA (n_components=100) and you need IC gene loadings on a comparable scale with consistently oriented (positively skewed) directions before downstream clustering.

## When NOT to use

- Input is already a normalized loading/signature table or annotated feature set — no decomposition or sign alignment is needed.
- Goal is annotating new TME cells using MetaTiME's pretrained meta-components (MetaTiME-Annotator projection), not re-learning components from raw data.
- Expression matrix is not log-transformed single-cell TME data (e.g., bulk RNA-seq), where the ICA-based MetaTiME decomposition does not apply.

## Inputs

- Log-transformed tumor scRNA-seq expression matrix (genes x cells), loadable via metatime.loaddata helpers or the packaged example data in metatime.testdata
- Number of independent components to extract (MetaTiME uses n_components=100)

## Outputs

- ic_loadings.tsv: processed loading matrix (genes x 100 ICs) with z-weight-normalized, positively skewed component loadings

## How to apply

Run fastICA on a log-transformed tumor scRNA-seq expression matrix to obtain n_components=100 maximally independent gene-expression components (MetaTiME uses ICA to optimize information independence among components). Z-score-transform the gene loading values of each component so loadings are comparable across datasets and components. Compute the skewness of each component's loading distribution and flip the component's sign wherever skewness is negative, so all directions are positively skewed. Assemble the resulting genes x 100 ICs loading matrix and write it to ic_loadings.tsv for downstream clustering.

## Related tools

- **MetaTiME** (Python package (pip install metatime) providing the ICA-based decomposition pipeline, loaddata/testdata helpers, and the z-weight normalization and skewness alignment transforms applied to IC loadings) — https://github.com/yi-zhang/MetaTiME

## Examples

```
from sklearn.decomposition import FastICA; import scipy.stats as st; ica = FastICA(n_components=100); L = ica.fit_transform(X.T).T; L = (L - L.mean(0)) / L.std(0); L[:, st.skew(L) < 0] *= -1
```

## Evaluation signals

- Output file ic_loadings.tsv exists after the run
- Output loading matrix has exactly 100 component columns (matching n_components=100)
- Component loading distributions are predominantly positively skewed after sign alignment
- Each component's loadings are z-scored (comparable scale across components), making gene representation scores consistent across datasets

## Limitations

- Sign flipping is driven purely by skewness convention; a component's biological orientation is fixed by this rule rather than by prior biological annotation.
- The decomposition is defined at 100 components for tumor scRNA-seq; the number of components and applicability to non-TME data are not established by this pipeline.
- The repo is 'continously being improved'; dependency versions are pinned (e.g., pandas==1.1.5, scanpy==1.8.2, anndata==0.8.0), so newer environments may break the helpers.

## Evidence

- [discussion] performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51.: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51."
- [abstract] Using millions of TME single cells, MetaTiME learns meta-components that encode independent components of gene expression observed across cancer types.: "Using millions of TME single cells, MetaTiME learns meta-components that encode independent components of gene expression observed across cancer types."
- [readme] MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data.: "MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data."
