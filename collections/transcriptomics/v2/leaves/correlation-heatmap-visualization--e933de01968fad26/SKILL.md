---
name: correlation-heatmap-visualization
description: Use when comparing cell-abundance deconvolution methods on pseudo-bulk
  single-cell data where true per-cell-type fractions are known, and results must
  be visualized as a per-cell-type Pearson correlation heatmap across methods and
  datasets.
when_to_use_negative:
- Input data lack ground-truth cell abundances (e.g., bulk tumor samples like TCGA
  where correlations against true fractions cannot be computed)
- The goal is deconvolution itself rather than benchmarking/visualizing method agreement
  with known abundances
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3518
tools:
- name: DCNet
  role: Deep-learning deconvolution method whose predicted cell abundances are correlated
    against true abundances and baseline methods
  repo: https://github.com/xindd/DCNet
- name: CIBERSORTx
  role: Baseline deconvolution method included as a column in the correlation heatmap
- name: EPIC
  role: Baseline deconvolution method included as a column in the correlation heatmap
- name: MCP-counter
  role: Baseline deconvolution method included as a column in the correlation heatmap
- name: quanTIseq
  role: Baseline deconvolution method included as a column in the correlation heatmap
- name: TIMER
  role: Baseline deconvolution method included as a column in the correlation heatmap
- name: xCell
  role: Baseline deconvolution method included as a column in the correlation heatmap
- name: Python
  role: Language used (with mxnet) to run DCNet and compute correlations / render
    the heatmap
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:fbe03e42f0a423f1d6588892439ee8603626d3f960b43240c33ad4d6d6d0afb6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - CIBERSORTx
  - DCNet
  - EPIC
  - MCP-counter
  - Python
  - TIMER
  - quanTIseq
  - xCell
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# correlation-heatmap-visualization

## Summary

Assembles per-cell-type Pearson correlations between predicted and true cell abundances (e.g., DCNet vs. CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, xCell, Marker_sum) into a heatmap reproducing Figure 3a-style benchmark comparisons on CRC (GSE81861) and melanoma (GSE72056) pseudo-bulk samples.

## When to use

Use when comparing cell-abundance deconvolution methods on pseudo-bulk single-cell data where true per-cell-type fractions are known, and results must be visualized as a per-cell-type Pearson correlation heatmap across methods and datasets.

## When NOT to use

- Input data lack ground-truth cell abundances (e.g., bulk tumor samples like TCGA where correlations against true fractions cannot be computed)
- The goal is deconvolution itself rather than benchmarking/visualizing method agreement with known abundances

## Inputs

- Pseudo-bulk expression matrix (rows = human gene symbols, columns = samples) aggregated from GSE81861 (CRC, 7 cell types) and GSE72056 (melanoma, 6 cell types)
- Known true per-cell-type abundances for each pseudo-bulk sample
- Per-method predicted cell abundance tables (DCNet, CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, xCell, Marker_sum)

## Outputs

- Per-cell-type Pearson correlation matrix (methods x cell types, per dataset)
- Correlation heatmap image, e.g. figure3a_correlation_heatmap.png

## How to apply

Aggregate single-cell GEO datasets (GSE81861 CRC with 7 cell types; GSE72056 melanoma with 6 cell types) into pseudo-bulk matrices with rows as human gene symbols and columns as samples, and apply the recommended log2(x+1) transformation. Run DCNet using index.pkl gene sets for input/output neurons and the zero-one gene-cell relation matrix (rix.pkl), then run CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, and xCell with default parameters and compute Marker_sum by summing marker-gene expression per cell type. Restrict each method's predictions to the cell types present in each dataset, compute per-cell-type Pearson correlations between predicted and known true abundance, and render the correlation matrix for all methods and both datasets as a heatmap (figure3a_correlation_heatmap.png). The rationale is that DCNet's intermediate-layer activations should show stable correlations (>0.7) across all cell types, while baselines can deviate or even anti-correlate.

## Related tools

- **DCNet** (Deep-learning deconvolution method whose predicted cell abundances are correlated against true abundances and baseline methods) — https://github.com/xindd/DCNet
- **CIBERSORTx** (Baseline deconvolution method included as a column in the correlation heatmap)
- **EPIC** (Baseline deconvolution method included as a column in the correlation heatmap)
- **MCP-counter** (Baseline deconvolution method included as a column in the correlation heatmap)
- **quanTIseq** (Baseline deconvolution method included as a column in the correlation heatmap)
- **TIMER** (Baseline deconvolution method included as a column in the correlation heatmap)
- **xCell** (Baseline deconvolution method included as a column in the correlation heatmap)
- **Python** (Language used (with mxnet) to run DCNet and compute correlations / render the heatmap)

## Examples

```
import pandas as pd, numpy as np; r = {ct: np.corrcoef(pred[ct], true[ct])[0,1] for ct in types}; pd.DataFrame(r).T.plotheatmap('figure3a_correlation_heatmap.png')
```

## Evaluation signals

- Reproduced heatmap matches Figure 3a's pattern: DCNet T-cell and B-cell Pearson correlations of 0.81 and 0.95 in melanoma
- xCell macrophage correlation is -0.09 in CRC and Marker_sum B-cell correlation is -0.13 in CRC
- All DCNet per-cell-type correlations exceed 0.7 in both CRC and melanoma pseudo-bulk samples
- Each method's predictions are restricted to the cell types present in the corresponding dataset before correlation

## Limitations

- Baseline methods are run with default parameters only, so the comparison reflects untuned settings
- Correlations are only computable where true abundances are known; pseudo-bulk aggregation of GEO single-cell data is required
- Each method covers a different subset of cell types, so heatmap columns must be restricted to overlapping cell types per dataset

## Evidence

- [discussion] baseline comparison methods: [evidence span withheld — anchor status: not_found]
