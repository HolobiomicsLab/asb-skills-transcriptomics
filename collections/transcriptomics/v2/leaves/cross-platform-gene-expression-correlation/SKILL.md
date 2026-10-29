---
name: cross-platform-gene-expression-correlation
description: Use when a cell-gene expression matrix from segmented spatial transcriptomics
  data needs benchmarking against a matched scRNA-seq reference (e.g. Chromium scFFPE-seq
  of the same tissue) via expression-profile correlation.
when_to_use_negative:
- The spatial data lack a paired single-cell reference dataset from the same tissue,
  so no matched average-expression profile can be computed.
- 'The task is pure segmentation generation (run_pipeline: preprocess, train, predict)
  rather than evaluation of an already-produced expression matrix.'
- Cell-type identity annotation (not expression-profile concordance) is the target;
  neighbour-contamination distance analyses (Fig. 3e) would be more appropriate.
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_3170
tools:
- name: BIDCell
  role: Produces the cell-gene expression matrix (expr_mat.csv) via run_pipeline /
    make_cell_gene_mat, which is the input to the correlation evaluation
  repo: https://github.com/SydneyBioX/BIDCell
- name: 10x Genomics Xenium
  role: Platform generating the Xenium-BreastCancer1 subcellular spatial transcriptomics
    data being evaluated
- name: TISCH2
  role: Public repository from which single-cell reference data (sc_breast.csv) for
    average expression per cell type are obtained
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:d6cf6ffd9ac5f269cca24db47e220f25e13f588b74cefeda080137f9814bd68b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - 10x Genomics Xenium
  - BIDCell
  - TISCH2
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cross-platform-gene-expression-correlation

## Summary

Evaluates a subcellular spatial transcriptomics segmentation (e.g. BIDCell on Xenium-BreastCancer1) by correlating its average cell-gene expression profile with the average expression from a paired Chromium scFFPE-seq single-cell reference, as in Fig. 3a-d of the BIDCell paper.

## When to use

Use when a cell-gene expression matrix from segmented spatial transcriptomics data needs benchmarking against a matched scRNA-seq reference (e.g. Chromium scFFPE-seq of the same tissue) via expression-profile correlation.

## When NOT to use

- The spatial data lack a paired single-cell reference dataset from the same tissue, so no matched average-expression profile can be computed.
- The task is pure segmentation generation (run_pipeline: preprocess, train, predict) rather than evaluation of an already-produced expression matrix.
- Cell-type identity annotation (not expression-profile concordance) is the target; neighbour-contamination distance analyses (Fig. 3e) would be more appropriate.

## Inputs

- BIDCell cell-gene expression matrix (expr_mat.csv) from segmented Xenium-BreastCancer1 data
- Breast cancer single-cell reference sc_breast.csv (TISCH-BRCA / Chromium scFFPE-seq average expression per cell type)
- Positive and negative marker files sc_breast_markers_pos.csv and sc_breast_markers_neg.csv

## Outputs

- Cross-platform correlation values between segmented-cell average expression and Chromium scFFPE-seq average expression (Fig. 3a-d)
- Positive and negative purity F1 scores per evaluation category
- fig3_evaluation_metrics.csv containing correlation and purity F1 metrics

## How to apply

Load the segmentation-derived cell-gene expression matrix (e.g. expr_mat.csv from BIDCell's model.make_cell_gene_mat()) with pandas, together with the paired single-cell reference (TISCH-BRCA / sc_breast.csv). Compute the correlation between the average gene expression of the segmented cells and the average expression from the paired Chromium scFFPE-seq data, per marker panel or evaluation category (Fig. 3a-d). Optionally complement this with positive/negative purity F1 scores, defining markers as genes in the highest and lowest 10 percentile per cell type and excluding positive markers common to at least a third of cell types. Write all metrics (correlation and purity F1 values) to a summary CSV (e.g. fig3_evaluation_metrics.csv) and check that values are consistent with the relationships reported across evaluation categories in Fig. 3a-d.

## Related tools

- **BIDCell** (Produces the cell-gene expression matrix (expr_mat.csv) via run_pipeline / make_cell_gene_mat, which is the input to the correlation evaluation) — https://github.com/SydneyBioX/BIDCell
- **10x Genomics Xenium** (Platform generating the Xenium-BreastCancer1 subcellular spatial transcriptomics data being evaluated)
- **TISCH2** (Public repository from which single-cell reference data (sc_breast.csv) for average expression per cell type are obtained)

## Examples

```
from bidcell import BIDCellModel; model = BIDCellModel('xenium_example_config.yaml'); model.run_pipeline()  # then correlate expr_mat.csv vs sc_breast.csv in pandas
```

## Evaluation signals

- Correlation between average segmented-cell expression and paired Chromium scFFPE-seq average expression is consistent across evaluation categories as shown in Fig. 3a-d.
- Positive and negative purity F1 scores are computed with markers defined as genes in the highest and lowest 10 percentile per cell type.
- Positive markers common to at least a third of cell types are excluded before scoring, matching the paper's filtering rule.
- All computed metrics (correlation, purity F1 scores) are present in fig3_evaluation_metrics.csv with the expected categories.

## Limitations

- Manual annotation of ground truth is unattainable for SST data containing hundreds of molecular units per pixel, so correlation against a reference is an indirect quality measure.
- Removing positive markers common to at least a third of cell types was found appropriate for these datasets, but larger numbers of positive markers tend to increase the size of predicted cells.
- Only <1,000 genes may be needed for segmentation, so correlation coverage is limited to the marker panel of the spatial platform (e.g. Xenium's panel).

## Evidence

- [methods] Compute the correlation between the average gene expression of BIDCell-segmented cells and the average expression from the paired Chromium scFFPE-seq data (Fig. 3a-d).: [evidence span withheld — anchor status: not_found]
- [readme] The positive and negative markers were those with expressions in the highest and lowest 10 percentile for each cell type of a tissue sample.: "The positive and negative markers were those with expressions in the highest and lowest 10 percentile for each cell type of a tissue sample."
- [readme] We found that removing positive markers that were common to at least a third of cell types in each dataset was appropriate across various datasets.: "We found that removing positive markers that were common to at least a third of cell types in each dataset was appropriate across various datasets."
- [readme] Please see the provided breast cancer single-cell reference and positive/negative marker files (`sc_breast.csv`, `sc_breast_markers_pos.csv`, and `sc_breast_markers_neg.csv`): "Please see the provided breast cancer single-cell reference and positive/negative marker files (`sc_breast.csv`, `sc_breast_markers_pos.csv`, and `sc_breast_markers_neg.csv`)"
- [readme] model.make_cell_gene_mat(is_cell=False): "model.make_cell_gene_mat(is_cell=False)"
