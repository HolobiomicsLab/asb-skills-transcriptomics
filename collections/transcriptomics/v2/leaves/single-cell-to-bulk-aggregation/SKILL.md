---
name: single-cell-to-bulk-aggregation
description: Use when raw single-cell GEO datasets (e.g. GSE81861 CRC, GSE72056 melanoma,
  GSE86146 embryonic) must be converted into bulk-aggregated expression matrices to
  validate cell-abundance inference or run bulk deconvolution baselines.
when_to_use_negative:
- The input is already bulk RNA-seq (e.g. TCGA samples downloaded from GDC) — no aggregation
  is needed, only the log2(x+1) transform
- The downstream method expects single-cell-level granularity rather than sample-level
  pseudo-bulk profiles
- The analysis is the DCNet training pipeline itself on TCGA data, where the article
  instead splits samples 80/20 into train/test sets rather than aggregating single
  cells
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: DCNet
  role: Downstream deconvolution network consuming the aggregated, log2(x+1)-transformed
    pseudo-bulk matrix to infer cell-type abundances
  repo: https://github.com/xindd/DCNet
- name: CIBERSORTx
  role: Baseline deconvolution method run on the aggregated matrices with default
    parameters for correlation comparison
- name: EPIC
  role: Baseline deconvolution method run on the aggregated matrices with default
    parameters
- name: MCP-counter
  role: Baseline deconvolution method run on the aggregated matrices with default
    parameters
- name: quanTIseq
  role: Baseline deconvolution method run on the aggregated matrices with default
    parameters
- name: TIMER
  role: Baseline deconvolution method run on the aggregated matrices with default
    parameters
- name: xCell
  role: Baseline deconvolution method run on the aggregated matrices with default
    parameters
- name: Python
  role: Implementation environment for aggregation, the log2(x+1) transform, and DCNet
    (mxnet framework)
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:3fac0f2e7926aa075a19da277340cfa1015a983651e11f322a1cdea7928f9055
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-to-bulk-aggregation

## Summary

Aggregates single-cell RNA-seq profiles into pseudo-bulk expression matrices (rows = human gene symbols, columns = samples) so that bulk deconvolution methods such as DCNet, CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, and xCell can be benchmarked against known true cell abundances.

## When to use

Use when raw single-cell GEO datasets (e.g. GSE81861 CRC, GSE72056 melanoma, GSE86146 embryonic) must be converted into bulk-aggregated expression matrices to validate cell-abundance inference or run bulk deconvolution baselines.

## When NOT to use

- The input is already bulk RNA-seq (e.g. TCGA samples downloaded from GDC) — no aggregation is needed, only the log2(x+1) transform
- The downstream method expects single-cell-level granularity rather than sample-level pseudo-bulk profiles
- The analysis is the DCNet training pipeline itself on TCGA data, where the article instead splits samples 80/20 into train/test sets rather than aggregating single cells

## Inputs

- Single-cell RNA-seq expression matrices from GEO (e.g. GSE81861 CRC, GSE72056 melanoma, GSE86146 embryonic PGC/niche-cell data)
- Cell-type labels for each single cell (e.g. 7 CRC cell types, 6 melanoma cell types, FGC vs somatic cells)
- DCNet input specification: index.pkl gene sets and rix.pkl zero-one gene-cell correspondence matrix

## Outputs

- Pseudo-bulk expression matrix with rows as human gene symbols and columns as samples
- log2(x+1)-transformed expression matrix suitable for DCNet and deconvolution baselines
- Known true per-cell-type abundance vector per aggregated sample for correlation validation

## How to apply

Download the single-cell expression data from GEO, then aggregate single-cell profiles into pseudo-bulk samples per dataset/cell type (e.g. 11 CRC samples with 7 cell types from GSE81861, 31 melanoma samples with 6 cell types from GSE72056, and 2167 PGCs plus gonadal niche cells from GSE86146). Format the matrix with each column as a sample and each row as a human gene symbol, matching DCNet's required input layout. Apply the recommended log2(x+1) transformation to the expression values before feeding the matrix to DCNet (along with index.pkl for gene-to-neuron mapping and rix.pkl/relation_matrix.pkl for gene-cell correspondence) or to the comparison baselines run with default parameters. For validation datasets, retain the true per-cell-type abundances computed from the aggregation so Pearson correlations between predicted and true abundance can be computed per cell type.

## Related tools

- **DCNet** (Downstream deconvolution network consuming the aggregated, log2(x+1)-transformed pseudo-bulk matrix to infer cell-type abundances) — https://github.com/xindd/DCNet
- **CIBERSORTx** (Baseline deconvolution method run on the aggregated matrices with default parameters for correlation comparison)
- **EPIC** (Baseline deconvolution method run on the aggregated matrices with default parameters)
- **MCP-counter** (Baseline deconvolution method run on the aggregated matrices with default parameters)
- **quanTIseq** (Baseline deconvolution method run on the aggregated matrices with default parameters)
- **TIMER** (Baseline deconvolution method run on the aggregated matrices with default parameters)
- **xCell** (Baseline deconvolution method run on the aggregated matrices with default parameters)
- **Python** (Implementation environment for aggregation, the log2(x+1) transform, and DCNet (mxnet framework))

## Examples

```
import pandas as pd, numpy as np; bulk = sc_expr.groupby(cell_labels).sum().T; bulk_t = np.log2(bulk + 1)
```

## Evaluation signals

- Aggregated matrix shape/layout check: each column is a sample and each row is a human gene symbol, matching DCNet's stated input requirement
- After aggregation, per-cell-type Pearson correlations of DCNet predictions vs true abundance should exceed 0.7 across all cell types (e.g. T/B cell correlations of 0.81/0.95 in melanoma; 0.87 FGC and 0.79 somatic in GSE86146, p < .01)
- Negative-control correlations should reproduce known baseline failures: xCell macrophage correlation -0.09 in CRC and Marker_sum B cell correlation -0.13 in CRC
- Verify the log2(x+1) transformation was applied before running DCNet, as recommended in the methods
- Saved result tables (e.g. gse86146_correlations.csv) should report correlations significant at p < 0.01 against true abundances derived from the aggregation

## Limitations

- Baseline comparison methods were only run with default parameters ('Each method is set to default parameters.'), so aggregation-benchmark results may not reflect tuned performance
- No random seed or reproducibility controls are reported for the downstream model, and trained model weights/checkpoints are not stated as available — only source code
- Only 7 of the drug-treatment records were retained after a 48-patient threshold, illustrating how aggressive filtering after aggregation can shrink usable samples
- Gene filtering (deleting genes with zero expression in more than 1/3 of TCGA samples, retaining 21,136 genes) must be accounted for so the aggregated matrix contains genes detectable in the target gene universe

## Evidence

- [results] Validation benchmark values from source cards: [evidence span withheld — anchor status: not_found]
- [results] Non-immune cell validation on GSE86146: [evidence span withheld — anchor status: not_found]
