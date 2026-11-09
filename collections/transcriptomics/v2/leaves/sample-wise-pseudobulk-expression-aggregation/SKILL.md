---
name: sample-wise-pseudobulk-expression-aggregation
description: Use when you have an adjusted single-cell expression matrix from multiple
  samples and conditions and need valid sample-level differential state testing per
  cell type rather than naive per-cell differential expression.
when_to_use_negative:
- The integration method returns only a low-dimensional embedding and no adjusted
  gene expression matrix, which the paper notes restricts downstream sample-level
  analysis
- Data are atlas-scale and you only need a consensus cell-type map rather than differential
  state testing between conditions
- Analysis is per-cell (clustering, visualisation) rather than sample-level differential
  testing
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scMerge2
  role: Produces the adjusted expression matrix whose per-cell values are aggregated
    into sample-wise pseudo-bulk profiles
  repo: https://github.com/SydneyBioX/scMerge
- name: limma
  role: Fits cell-type-specific linear models on the sample-wise aggregated data (limma-trend,
    eBayes trend=TRUE) to call DS genes
- name: Seurat
  role: Alternative integration method producing adjusted matrices benchmarked through
    the same pseudo-bulk DS-detection pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:40fba13afc66068899d925855e4c669ce38fa12013aa79d18fbd6e945d1b29bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - BBKNN
  - Harmony
  - SAUCIE
  - Scanorama
  - Seurat
  - deepMNN
  - limma
  - scANVI
  - scMerge2
  - scVI
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# sample-wise-pseudobulk-expression-aggregation

## Summary

Aggregate per-cell adjusted expression (e.g. from scMerge2-integrated matrices) into sample- and cell-type-level pseudo-bulk profiles so samples become replication units, then fit cell-type-specific limma-trend models to call differential state genes and score them (F1, FDR, TPR) against known ground truth.

## When to use

Use when you have an adjusted single-cell expression matrix from multiple samples and conditions and need valid sample-level differential state testing per cell type rather than naive per-cell differential expression.

## When NOT to use

- The integration method returns only a low-dimensional embedding and no adjusted gene expression matrix, which the paper notes restricts downstream sample-level analysis
- Data are atlas-scale and you only need a consensus cell-type map rather than differential state testing between conditions
- Analysis is per-cell (clustering, visualisation) rather than sample-level differential testing

## Inputs

- Adjusted/integrated single-cell expression matrix (e.g. scMerge2 output on simulated breast-cancer-base multi-sample multi-condition scRNA-seq data)
- Per-cell metadata mapping cells to sample, condition, and cell type
- Ground-truth DS gene set (simulation setting) for F1/FDR/TPR evaluation

## Outputs

- Per-sample-per-cell-type pseudo-bulk expression matrix
- Cell-type-specific DS gene calls from limma-trend (eBayes trend=TRUE)
- Performance metrics (F1-score, FDR, TPR) per method and logFC setting, e.g. rendered as FigS7_v1.png

## How to apply

First run each integration method that returns an adjusted expression matrix (scMerge2, Seurat, Scanorama, Harmony, BBKNN, deepMNN, scVI, scANVI, SAUCIE) so cells occupy a shared expression space. For each cell type, sum or average the adjusted expression across all cells belonging to the same sample and the same cell type, creating a pseudo-bulk profile per sample per cell type so samples — not cells — are the replication units. Fit cell-type-specific linear models on these sample-wise aggregated profiles using limma-trend (eBayes with trend=TRUE) to call differential state genes between conditions. Rationale: pseudo-bulk building both minimises processing demand and avoids pseudo-replication inflated by treating individual cells as independent replicates. In the benchmark setting, compare called DS genes against injected ground truth across a grid of log-fold-change magnitudes and compute F1-score, FDR and TPR per method and setting.

## Related tools

- **scMerge2** (Produces the adjusted expression matrix whose per-cell values are aggregated into sample-wise pseudo-bulk profiles) — https://github.com/SydneyBioX/scMerge
- **limma** (Fits cell-type-specific linear models on the sample-wise aggregated data (limma-trend, eBayes trend=TRUE) to call DS genes)
- **Seurat** (Alternative integration method producing adjusted matrices benchmarked through the same pseudo-bulk DS-detection pipeline)

## Examples

```
fit <- eBayes(lmFit(pb.exprs, design = model.matrix(~condition, pb.meta)), trend = TRUE)
```

## Evaluation signals

- The pseudo-bulk matrix has exactly one row per sample per cell type, with samples as replication units
- limma models are fitted with eBayes(trend=TRUE) per cell type on sample-level (not cell-level) values
- In simulation, F1-score rises and FDR falls with larger injected logFC magnitudes, and scMerge2 shows lower FDR and higher TPR than fastMNN, Seurat and raw unadjusted data
- Metrics reproduce the F1/FDR/TPR patterns reported in Fig. 2c and Suppl. Figs. S6–S8
- Output figure file FigS7_v1.png exists and renders correctly

## Limitations

- Cells within a sample must not be treated as independent replicates; pseudo-replication is required to account for circumstances with phenotypes
- Only methods returning adjusted expression matrices can be evaluated this way; embedding-only methods increase interpretability challenges since a low-dimensional representation does not lend itself to such downstream features
- Multi-modal integration was only performed within each modality; joint use of multi-modal information is deferred to future work

## Evidence

- [discussion] pseudo-bulk building and pseudo-replication rationale: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
- [intro] lack of adjusted matrices restricts downstream use: "the lack of adjusted expression matrices restricts the utilisation of such integrative results and diminishes their potency and generalizability"
- [full_text] pseudo-bulk aggregation step from the reproduction workflow: [evidence span withheld — anchor status: not_found]
- [full_text] limma-trend DS calling step: [evidence span withheld — anchor status: not_found]
