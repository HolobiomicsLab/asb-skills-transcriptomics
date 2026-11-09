---
name: cross-dataset-component-comparability-handling
description: Use when decomposing individual tumor scRNA-seq expression matrices with
  ICA (n_components=100) and you need gene loading scales and orientations comparable
  across components and datasets before downstream clustering or pan-cancer integration.
when_to_use_negative:
- You only need to annotate new TME cells using MetaTiME's pretrained meta-components
  (MeCs) rather than re-deriving components from a new dataset
- Your goal is joint low-dimensional embedding across datasets with algorithms like
  Harmony or scArches/scVI rather than per-dataset ICA decomposition with normalized
  loadings
- The input is already a normalized feature/signature table rather than a raw log-transformed
  expression matrix needing decomposition
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MetaTiME
  role: Provides the metatime.loaddata/testdata data helpers and the pipeline in which
    fastICA decomposition, z-weight normalization, and skewness alignment produce
    comparable IC loadings across tumor scRNA-seq datasets
  repo: https://github.com/yi-zhang/MetaTiME
- name: fastICA
  role: Independent component analysis used to decompose each expression matrix into
    n_components=100 maximally independent gene-expression components
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:4ef22878958809ed69ec8cdc1a37cac34935666daa1d975fa9b9043d9881af6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - MetaTiME
  - fastICA
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cross-dataset-component-comparability-handling

## Summary

A normalization-and-alignment procedure from MetaTiME that makes fastICA-derived gene loading vectors comparable across tumor scRNA-seq datasets, via per-component z-weight normalization of loadings and positive-skewness sign alignment, yielding a standardized genes x 100-IC loading matrix.

## When to use

Use when decomposing individual tumor scRNA-seq expression matrices with ICA (n_components=100) and you need gene loading scales and orientations comparable across components and datasets before downstream clustering or pan-cancer integration.

## When NOT to use

- You only need to annotate new TME cells using MetaTiME's pretrained meta-components (MeCs) rather than re-deriving components from a new dataset
- Your goal is joint low-dimensional embedding across datasets with algorithms like Harmony or scArches/scVI rather than per-dataset ICA decomposition with normalized loadings
- The input is already a normalized feature/signature table rather than a raw log-transformed expression matrix needing decomposition

## Inputs

- Log-transformed tumor scRNA-seq expression matrix (genes x cells), loadable via metatime.loaddata helpers or metatime.testdata example data

## Outputs

- Normalized IC loading matrix (genes x 100 independent components) with z-weight-normalized, positively-skew-aligned loadings
- ic_loadings.tsv file containing the processed loading matrix

## How to apply

Load a log-transformed tumor scRNA-seq expression matrix via the metatime.loaddata helpers (or metatime.testdata example data) and run fastICA to decompose it into n_components=100 maximally independent gene-expression components. For each component, z-score-transform the gene loading values so loading scales are comparable across datasets, reflecting MetaTiME's premise that meta-components must be reproducible across cohorts and cancer types. Then check each component's skewness and flip its sign so the skewness is positive, ensuring consistently oriented transcriptional directions. Assemble the resulting genes x 100 IC loading matrix and write it to ic_loadings.tsv. This mirrors the paper's large-scale pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components.

## Related tools

- **MetaTiME** (Provides the metatime.loaddata/testdata data helpers and the pipeline in which fastICA decomposition, z-weight normalization, and skewness alignment produce comparable IC loadings across tumor scRNA-seq datasets) — https://github.com/yi-zhang/MetaTiME
- **fastICA** (Independent component analysis used to decompose each expression matrix into n_components=100 maximally independent gene-expression components)

## Examples

```
from sklearn.decomposition import FastICA; ica = FastICA(n_components=100); loadings = ica.fit_transform(expr_df.T)  # then z-score and flip to positive skew; save ic_loadings.tsv
```

## Evaluation signals

- ic_loadings.tsv exists and contains exactly 100 component columns
- Per-component gene loading distributions are approximately standard-normal (z-weight normalized) so scales are comparable among components
- After sign alignment, component loading distributions are predominantly positively skewed
- Loadings from different tumor scRNA-seq datasets can be directly compared in downstream clustering without rescaling, reflecting reproducible components across cohorts and cancer types

## Limitations

- Normalization is per-dataset ICA-based; the paper notes prior cell type definitions based on representative genes were chosen differently across studies, motivating but also bounding what comparability fixes
- The pretrained MeC space comprises 86 interpretable meta-components, so datasets decomposed into 100 ICs are not guaranteed one-to-one with the published MeC annotations
- MetaTiME is scoped to tumor immune microenvironment (TME) scRNA-seq data; applicability to other tissue or non-single-cell data is not established in the provided context

## Evidence

- [intro] MetaTiME identifies reproducible low-dimensional meta-components: [evidence span withheld — anchor status: not_found]
- [discussion] pan-cancer integration using ICA: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51."
- [abstract] MeCs encode independent components across cancer types: "Using millions of TME single cells, MetaTiME learns meta-components that encode independent components of gene expression observed across cancer types."
- [readme] pretrained MeCs for annotation: "MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment."
