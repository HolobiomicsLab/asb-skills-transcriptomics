---
name: single-cell-expression-matrix-loading
description: Use when a log-transformed single-cell expression matrix from a tumor
  microenvironment sample must be reduced to independent gene-expression components
  whose loading scores are comparable across components and datasets.
when_to_use_negative:
- The goal is annotating new TME cells with existing signatures — for that, use the
  pretrained MetaTiME-Annotator MeC projection rather than re-decomposing the matrix
- The input is already a dimensionality-reduced or integrated feature embedding (e.g.
  Harmony or scVI latent space), not a gene-by-cell expression matrix
- The data are not single-cell TME expression data and no gene-loadings-per-dataset
  output is needed
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MetaTiME
  role: Framework providing the loaddata/testdata helpers and the ICA-based meta-component
    discovery pipeline applied to each tumor scRNA-seq dataset
  repo: https://github.com/yi-zhang/MetaTiME
- name: fastICA
  role: Decomposes the expression matrix into n_components=100 maximally independent
    gene-expression components
- name: scanpy
  role: Dependency for reading and handling single-cell expression data (anndata objects)
    prior to decomposition
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:469aca6a1b4e54446eb9cc0b37d82fbb1ff8d53639339a6834110cdcfaa72800
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yi-zhang/MetaTiME
  tools:
  - MetaTiME
  - fastICA
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-expression-matrix-loading

## Summary

Loads a log-transformed tumor scRNA-seq expression matrix into MetaTiME and decomposes it via fastICA into 100 independent gene-expression components, then normalizes IC loadings (z-weight normalization plus positive-skewness sign alignment) so component scales are comparable across datasets.

## When to use

Use when a log-transformed single-cell expression matrix from a tumor microenvironment sample must be reduced to independent gene-expression components whose loading scores are comparable across components and datasets.

## When NOT to use

- The goal is annotating new TME cells with existing signatures — for that, use the pretrained MetaTiME-Annotator MeC projection rather than re-decomposing the matrix
- The input is already a dimensionality-reduced or integrated feature embedding (e.g. Harmony or scVI latent space), not a gene-by-cell expression matrix
- The data are not single-cell TME expression data and no gene-loadings-per-dataset output is needed

## Inputs

- Log-transformed tumor scRNA-seq expression matrix (e.g. loaded via metatime.loaddata from .h5ad or similar single-cell formats)
- Packaged example data via metatime.testdata
- Number of independent components to extract (n_components=100)

## Outputs

- Genes x 100 ICs normalized loading matrix written to ic_loadings.tsv
- Per-component sign-aligned, z-scored gene loading distributions

## How to apply

Load one log-transformed tumor scRNA-seq expression matrix through the metatime.loaddata helpers (or the packaged example data exposed by metatime.testdata). Run fastICA on the expression matrix to decompose it into n_components=100 maximally independent gene-expression components; ICA is used because MetaTiME optimizes information independence among components during its large-scale pan-cancer integration. Z-score-transform the gene loading values of each component so loadings are comparable across datasets. Check the skewness of each component and flip its sign so the skewness is positive, yielding consistently oriented transcriptional directions. Assemble the processed genes x 100 ICs loading matrix and write it to ic_loadings.tsv for downstream meta-component clustering and annotation.

## Related tools

- **MetaTiME** (Framework providing the loaddata/testdata helpers and the ICA-based meta-component discovery pipeline applied to each tumor scRNA-seq dataset) — https://github.com/yi-zhang/MetaTiME
- **fastICA** (Decomposes the expression matrix into n_components=100 maximally independent gene-expression components)
- **scanpy** (Dependency for reading and handling single-cell expression data (anndata objects) prior to decomposition)

## Examples

```
pip install metatime
```

## Evaluation signals

- The output file ic_loadings.tsv exists and is written successfully after the run
- ic_loadings.tsv contains exactly 100 component columns alongside the gene rows
- Component loading distributions are predominantly positively skewed after sign alignment
- Gene loading values are z-scored per component, so loading magnitudes are comparable across the 100 ICs
- Decomposition uses ICA (not PCA), consistent with MetaTiME's rationale of optimizing information independence among components

## Limitations

- The per-dataset ICA decomposition yields 100 fitted components, whereas MetaTiME's published MeC space comprises 86 interpretable meta-components; alignment to named MeCs is a downstream step
- Pretrained MeCs and training data are distributed via the separate MetaTiMEpretrain pipeline and Zenodo record (10.5281/zenodo.7410180); the pip-installed metatime package focuses on annotation of new data
- Tested dependency versions are specific (pandas==1.1.5, scanpy==1.8.2, anndata==0.8.0, matplotlib==3.5.1, adjustText==0.7.3, leidenalg==0.8.3); newer versions may break the pipeline
- Skewness-based sign alignment orients components but does not by itself assign biological meaning to each IC

## Evidence

- [discussion] Large-scale pan-cancer integration of tumor single cell datasets using ICA: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51."
- [intro] Reproducible independent components across cohorts: [evidence span withheld — anchor status: not_found]
- [readme] Pretrained MeCs for annotating new TME single cells: "MetaTiME provides pretrained meta-components (MeCs) to automatically annotate fine-grained cell states and plot signature continuum for new single-cells of tumor microenvironment."
