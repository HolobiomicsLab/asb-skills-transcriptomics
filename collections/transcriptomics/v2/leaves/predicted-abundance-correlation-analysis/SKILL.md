---
name: predicted-abundance-correlation-analysis
description: Use when bulk-aggregated single-cell samples with known per-cell-type
  abundances are available and you must quantify how faithfully a deconvolution method
  (e.g. DCNet, CIBERSORTx, xCell) recovers each cell type's abundance via per-type
  Pearson correlations.
when_to_use_negative:
- Input is raw single-cell data rather than pseudo-bulk-aggregated samples with known
  ground-truth abundances — there is no true abundance to correlate against
- 'You need a fully parameterized comparison: all baseline methods here are run with
  default parameters and no tuning, seed, or version control is reported'
- The tissue or cell types of interest are outside the datasets' coverage — DCNet's
  neuron/marker-gene mapping is fixed to 434 cells and 9078 marker genes embedded
  in the relation matrix
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: DCNet
  role: Deconvolution network whose intermediate-layer activations are scored for
    per-cell-type abundance correlation against true abundance
  repo: https://github.com/xindd/DCNet
- name: mxnet
  role: Python framework in which DCNet is implemented and run to infer cell abundances
  repo: https://github.com/xindd/DCNet
- name: CIBERSORTx
  role: Baseline deconvolution method run with default parameters for correlation
    comparison against DCNet
- name: EPIC
  role: Baseline deconvolution method run with default parameters for correlation
    comparison against DCNet
- name: MCP-counter
  role: Baseline deconvolution method run with default parameters for correlation
    comparison against DCNet
- name: quanTIseq
  role: Baseline deconvolution method run with default parameters for correlation
    comparison against DCNet
- name: TIMER
  role: Baseline deconvolution method run with default parameters for correlation
    comparison against DCNet
- name: xCell
  role: Baseline deconvolution method run with default parameters for correlation
    comparison against DCNet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:09fd8929d6e420440531c00453ebb09413307607562faf3521ba470d0b45c9f0
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
  - mxnet
  - quanTIseq
  - xCell
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# predicted-abundance-correlation-analysis

## Summary

Benchmark protocol for cell-type deconvolution methods (DCNet vs CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, xCell, Marker_sum) that scores per-cell-type Pearson correlation between predicted and true abundance on pseudo-bulk samples aggregated from GEO single-cell datasets (GSE81861 CRC, GSE72056 melanoma) and renders the results as a correlation heatmap.

## When to use

Use when bulk-aggregated single-cell samples with known per-cell-type abundances are available and you must quantify how faithfully a deconvolution method (e.g. DCNet, CIBERSORTx, xCell) recovers each cell type's abundance via per-type Pearson correlations.

## When NOT to use

- Input is raw single-cell data rather than pseudo-bulk-aggregated samples with known ground-truth abundances — there is no true abundance to correlate against
- You need a fully parameterized comparison: all baseline methods here are run with default parameters and no tuning, seed, or version control is reported
- The tissue or cell types of interest are outside the datasets' coverage — DCNet's neuron/marker-gene mapping is fixed to 434 cells and 9078 marker genes embedded in the relation matrix

## Inputs

- Pseudo-bulk expression matrix aggregated from GSE81861 (CRC) single-cell data: rows = human gene symbols, columns = samples
- Pseudo-bulk expression matrix aggregated from GSE72056 (melanoma) single-cell data: rows = human gene symbols, columns = samples
- Known true per-cell-type abundances for each pseudo-bulk sample (7 CRC cell types, 6 melanoma cell types)
- index.pkl gene sets for DCNet input/output neurons
- relation_matrix.pkl / rix.pkl zero-one gene-cell correspondence matrix
- DCNet trained parameters (net_M434_adam_T600_DCNetBnoise-D1.params)

## Outputs

- Per-cell-type Pearson correlation table of predicted vs true abundance for each method and dataset
- figure3a_correlation_heatmap.png rendering the per-cell-type correlation matrix across all methods and both datasets
- Per-method stability comparison (e.g. DCNet all correlations >0.7 vs xCell macrophage -0.09 in CRC)

## How to apply

Aggregate single-cell expression from GEO datasets (GSE81861 CRC, 7 cell types; GSE72056 melanoma, 6 cell types) into pseudo-bulk matrices with rows as human gene symbols and columns as samples, then apply the recommended log2(x+1) transformation. Run DCNet on the aggregated matrices using index.pkl (input/output neuron gene sets) and the zero-one gene-cell relation matrix (rix.pkl / relation_matrix.pkl), with trained parameters net_M434_adam_T600_DCNetBnoise-D1.params; run CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, and xCell with default parameters, and compute Marker_sum by summing marker-gene expression per cell type. Restrict each method's predictions to the cell types present in each dataset, then compute per-cell-type Pearson correlations between predicted and known true abundance. Assemble all method-by-dataset correlations into a single heatmap and compare against Figure 3a's reported pattern (e.g. DCNet T/B cell correlations of 0.81/0.95 in melanoma, all DCNet correlations >0.7). The rationale is that DCNet embeds 434 cell types and their marker genes in a hidden layer, whereas marker-gene-based baselines show unstable correlations that the heatmap exposes.

## Related tools

- **DCNet** (Deconvolution network whose intermediate-layer activations are scored for per-cell-type abundance correlation against true abundance) — https://github.com/xindd/DCNet
- **mxnet** (Python framework in which DCNet is implemented and run to infer cell abundances) — https://github.com/xindd/DCNet
- **CIBERSORTx** (Baseline deconvolution method run with default parameters for correlation comparison against DCNet)
- **EPIC** (Baseline deconvolution method run with default parameters for correlation comparison against DCNet)
- **MCP-counter** (Baseline deconvolution method run with default parameters for correlation comparison against DCNet)
- **quanTIseq** (Baseline deconvolution method run with default parameters for correlation comparison against DCNet)
- **TIMER** (Baseline deconvolution method run with default parameters for correlation comparison against DCNet)
- **xCell** (Baseline deconvolution method run with default parameters for correlation comparison against DCNet)

## Examples

```
expr = pd.read_pickle('demo_data.pkl'); expr_log = np.log2(expr + 1)  # rows=human gene symbols, cols=samples; then run DCNet per demo.ipynb with index.pkl and relation_matrix.pkl
```

## Evaluation signals

- Per-cell-type Pearson correlations reproduce Figure 3a's pattern: DCNet T/B cell correlations of 0.81/0.95 in melanoma, xCell macrophage correlation of -0.09 in CRC, Marker_sum B cell correlation of -0.13 in CRC
- All DCNet per-cell-type correlations exceed 0.7 on both CRC and melanoma pseudo-bulk samples
- Each method's predictions are restricted to the cell types present in the respective dataset (7 CRC, 6 melanoma) before correlation is computed
- Heatmap is rendered as figure3a_correlation_heatmap.png covering all methods (DCNet, Marker_sum, CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, xCell) and both datasets

## Limitations

- All comparison methods are run with default parameters and no parameter tuning is reported, so baseline performance may be suboptimal
- No software version numbers for mxnet or the comparison tools, no random seed, and no reproducibility controls for model training are reported; training was CPU-only with adam optimizer, relu activation, L2 loss, learning rate 1e-4, 600 iterations, 256 samples per batch
- The authors recommend choosing DCNet with a dropout of 0.1 and fine-tuning on user sample data, but the fine-tuning procedure is not specified
- Random gene dropout probabilities of 0.1/0.3/0.5 were used for augmentation but which setting produced the final model is not stated

## Evidence

- [discussion] we evaluated their cell abundance profiles by used CIBERSORTx: [evidence span withheld — anchor status: not_found]
