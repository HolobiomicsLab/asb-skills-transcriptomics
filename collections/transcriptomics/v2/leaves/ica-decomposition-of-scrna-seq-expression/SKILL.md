---
name: ica-decomposition-of-scrna-seq-expression
description: Use when you have a single log-transformed tumor scRNA-seq expression
  matrix and need reproducible, independent transcriptional directions (meta-components)
  rather than manual marker-based cell labeling.
when_to_use_negative:
- The goal is annotating cells against already-trained meta-components rather than
  re-deriving components — use the MetaTiME annotator projection workflow instead
- The input is not a single log-transformed expression matrix (e.g., it is already
  a feature/embedding table or multi-dataset joint matrix)
- You need to align datasets into a joint low-dimensional representation across cohorts
  — the paper contrasts this with Harmony and scVI/scArches integration approaches
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MetaTiME
  role: Framework whose meta-component discovery stage performs per-dataset fastICA
    decomposition with z-weight normalization and skewness alignment
  repo: github.com/yi-zhang/MetaTiME
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:dd5f800c01bbaed52d46d644cf596dc5044d6b361e711c8e9fa766c54d3e676a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaTiME
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ica-decomposition-of-scrna-seq-expression

## Summary

Decompose a log-transformed tumor scRNA-seq expression matrix into ~100 maximally independent gene-expression components via fastICA, then z-weight normalize and skewness-align the IC loading vectors so gene representation scores are comparable across components, yielding normalized IC loadings for downstream MeC clustering.

## When to use

Use when you have a single log-transformed tumor scRNA-seq expression matrix and need reproducible, independent transcriptional directions (meta-components) rather than manual marker-based cell labeling.

## When NOT to use

- The goal is annotating cells against already-trained meta-components rather than re-deriving components — use the MetaTiME annotator projection workflow instead
- The input is not a single log-transformed expression matrix (e.g., it is already a feature/embedding table or multi-dataset joint matrix)
- You need to align datasets into a joint low-dimensional representation across cohorts — the paper contrasts this with Harmony and scVI/scArches integration approaches

## Inputs

- Log-transformed tumor scRNA-seq expression matrix (genes x cells)
- metatime.loaddata helpers or packaged example data from metatime.testdata

## Outputs

- ic_loadings.tsv: normalized gene-loading matrix with genes as rows and 100 independent component columns
- Sign-aligned, z-score-normalized IC loading vectors used for downstream clustering

## How to apply

Load one log-transformed tumor scRNA-seq expression matrix through the metatime.loaddata helpers or the packaged example data in metatime.testdata. Run fastICA on the expression matrix to decompose it into n_components=100 maximally independent gene-expression components, following the paper's large-scale pan-cancer ICA integration strategy to optimize information independence among components. Z-score the gene loading values of each component so loadings are comparable across datasets and components. Check each component's skewness and flip its sign so the skewness is positive, yielding consistently oriented directions. Assemble the genes x 100 ICs loading matrix and write it to ic_loadings.tsv; these normalized loadings feed the downstream MeC clustering and annotation steps.

## Related tools

- **MetaTiME** (Framework whose meta-component discovery stage performs per-dataset fastICA decomposition with z-weight normalization and skewness alignment) — github.com/yi-zhang/MetaTiME

## Examples

```
from sklearn.decomposition import FastICA; ica = FastICA(n_components=100); loadings = ica.fit_transform(X)  # then z-score per component and flip signs so skewness is positive
```

## Evaluation signals

- ic_loadings.tsv exists and contains exactly 100 component columns
- Each component's gene loading values are z-score-normalized so loading scales are comparable across components and datasets
- Component loading distributions are predominantly positively skewed after sign alignment
- Decomposition yields maximally independent components (ICA objective converged) rather than correlated directions

## Limitations

- Per-dataset ICA produces dataset-specific components; cross-cohort consistency is only achieved after z-weight normalization and skewness alignment
- The published MeC space contains 86 interpretable meta-components while the per-dataset decomposition uses 100 ICs, so not every learned component maps to an annotated MeC
- The repository README states 'Repo continously being improved! More details will be updated and suggested improvements welcome.'

## Evidence

- [discussion] Large-scale pan-cancer integration rationale: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51."
- [intro] Meta-component definition: [evidence span withheld — anchor status: not_found]
- [readme] Pretrained MeC projection toolkit: "MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment."
